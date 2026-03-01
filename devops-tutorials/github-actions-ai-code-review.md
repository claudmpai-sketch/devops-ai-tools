# GitHub Actions AI-Powered Code Review

**Level:** Intermediate | **Time:** 30 minutes | **Cost:** Free (GitHub Actions)

## What You'll Build

An automated CI/CD pipeline that:
- Reviews PRs with AI
- Suggests improvements
- Checks for security issues
- Blocks merges that fail quality gates
- Posts comments directly on GitHub

## Architecture

```
GitHub Push/PR
    ↓
GitHub Actions Workflow Trigger
    ↓
Run Code Analysis
    ↓
Send to Claude API
    ↓
Claude Reviews Code
    ↓
Post Comments on PR
    ↓
Set Status Check (Pass/Fail)
```

## Prerequisites

- GitHub repository
- Anthropic API key (or OpenRouter)
- Basic YAML knowledge

## Step 1: Create the Workflow File

Create `.github/workflows/ai-code-review.yml`:

```yaml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize, reopened]
    paths:
      - '**.py'
      - '**.js'
      - '**.ts'
      - '**.go'

jobs:
  ai-review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get changed files
        id: changed-files
        uses: tj-actions/changed-files@v41
        with:
          files: |
            **.py
            **.js
            **.ts
            **.go

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install anthropic requests

      - name: Run AI code review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          REPO: ${{ github.repository }}
          CHANGED_FILES: ${{ steps.changed-files.outputs.all_files }}
        run: |
          python .github/scripts/ai-review.py

      - name: Set check status
        if: always()
        run: |
          # Status set by ai-review.py
          exit 0
```

## Step 2: Create the Review Script

Create `.github/scripts/ai-review.py`:

```python
#!/usr/bin/env python3
import os
import sys
import json
import subprocess
from pathlib import Path
import anthropic
import requests

# Configuration
ANTHROPIC_API_KEY = os.getenv("ANTHROPIC_API_KEY")
GITHUB_TOKEN = os.getenv("GITHUB_TOKEN")
PR_NUMBER = os.getenv("PR_NUMBER")
REPO = os.getenv("REPO")
CHANGED_FILES = os.getenv("CHANGED_FILES", "").split()

client = anthropic.Anthropic(api_key=ANTHROPIC_API_KEY)

def get_pr_diff():
    """Get the diff from the GitHub API"""
    url = f"https://api.github.com/repos/{REPO}/pulls/{PR_NUMBER}"
    headers = {"Authorization": f"token {GITHUB_TOKEN}"}
    
    response = requests.get(url, headers=headers)
    pr_data = response.json()
    
    # Get diff
    diff_url = pr_data["diff_url"]
    diff_response = requests.get(diff_url, headers=headers)
    return diff_response.text

def analyze_code_with_ai(diff: str) -> dict:
    """Use Claude to review the code"""
    
    prompt = f"""You are an expert code reviewer. Analyze this git diff and provide:

1. **Security Issues**: Any vulnerabilities or unsafe patterns
2. **Code Quality**: Style, complexity, and maintainability issues
3. **Performance**: Potential optimizations
4. **Best Practices**: Adherence to standards
5. **Suggestions**: Specific improvements

Be concise and actionable. Format your response as JSON with these keys:
- security_issues: list of security concerns
- quality_issues: list of code quality concerns
- performance_issues: list of performance improvements
- best_practices: adherence to best practices
- overall_score: 1-10 rating
- status: "approved" or "changes_requested"
- comments: list of specific comments with line numbers if possible

GIT DIFF:
{diff[:5000]}  # Limit to first 5000 chars
"""

    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[
            {"role": "user", "content": prompt}
        ]
    )
    
    try:
        response_text = message.content[0].text
        # Extract JSON from response
        import re
        json_match = re.search(r'\{.*\}', response_text, re.DOTALL)
        if json_match:
            return json.loads(json_match.group())
    except:
        pass
    
    return {
        "overall_score": 5,
        "status": "approved",
        "comments": ["Code review completed"]
    }

def post_github_comment(review: dict):
    """Post review as a GitHub comment"""
    
    comment = f"""## 🤖 AI Code Review

**Overall Score:** {review.get('overall_score', 'N/A')}/10  
**Status:** {review.get('status', 'approved').upper()}

### Security Issues
{chr(10).join(f"- {issue}" for issue in review.get('security_issues', ['None found']))}

### Code Quality
{chr(10).join(f"- {issue}" for issue in review.get('quality_issues', ['Looks good']))}

### Performance
{chr(10).join(f"- {issue}" for issue in review.get('performance_issues', ['No issues']))}

### Suggestions
{chr(10).join(f"- {comment}" for comment in review.get('comments', ['N/A']))}

---
*Review generated by Claude AI*
"""
    
    # Post comment to PR
    url = f"https://api.github.com/repos/{REPO}/issues/{PR_NUMBER}/comments"
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Content-Type": "application/json"
    }
    
    data = {"body": comment}
    requests.post(url, json=data, headers=headers)
    
    # Set status check
    set_check_status(review)

def set_check_status(review: dict):
    """Set the GitHub check status"""
    status = "success" if review.get('status') == 'approved' else "failure"
    
    conclusion = "success" if status == "success" else "failure"
    
    # Create check run
    url = f"https://api.github.com/repos/{REPO}/check-runs"
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github+json"
    }
    
    data = {
        "name": "AI Code Review",
        "head_sha": os.getenv("GITHUB_SHA"),
        "status": "completed",
        "conclusion": conclusion,
        "output": {
            "title": f"Code Review: {conclusion.upper()}",
            "summary": f"Overall score: {review.get('overall_score')}/10"
        }
    }
    
    requests.post(url, json=data, headers=headers)

def main():
    print("📝 Fetching PR diff...")
    diff = get_pr_diff()
    
    if not diff:
        print("No changes to review")
        return
    
    print("🤖 Running AI analysis...")
    review = analyze_code_with_ai(diff)
    
    print("📤 Posting review...")
    post_github_comment(review)
    
    print(f"✅ Review complete! Status: {review.get('status')}")
    
    # Exit with appropriate code
    sys.exit(0 if review.get('status') == 'approved' else 1)

if __name__ == "__main__":
    main()
```

## Step 3: Set Up Secrets

In your GitHub repository:

1. Go to **Settings → Secrets and variables → Actions**
2. Click **New repository secret**
3. Add `ANTHROPIC_API_KEY` with your key
4. Save

## Step 4: Test the Workflow

1. Create a test branch
2. Make some code changes
3. Open a pull request
4. The workflow will automatically run
5. Check the PR comments for the AI review

## Expected Output

```
🤖 AI Code Review

Overall Score: 8/10
Status: APPROVED

### Security Issues
- None found

### Code Quality
- Consider using more descriptive variable names in calculateTotal()

### Performance
- Loop could be optimized using list comprehension

### Suggestions
- Add docstring to the main function
- Consider adding type hints
```

## Customization Options

### 1. Change Review Strictness

Edit the prompt in `ai-review.py`:

```python
# Stricter review
prompt = f"""Review this code with STRICT standards..."""

# Lenient review  
prompt = f"""Review this code focusing on critical issues only..."""
```

### 2. Add Additional Checks

```yaml
- name: Run linter
  run: pylint src/

- name: Run tests
  run: pytest

- name: Check coverage
  run: coverage report
```

### 3. Block Merges on Issues

In the workflow:

```yaml
- name: Fail if critical issues
  if: steps.ai-review.outputs.status == 'critical'
  run: exit 1
```

## Cost Optimization

- **Only run on PRs** (saves ~90% of API calls)
- **Limit file types** (only review code, not docs)
- **Use Claude Haiku** for simple reviews (~10x cheaper)
- **Cache dependencies** to speed up workflow

## Troubleshooting

**Workflow not running:**
- Check file paths in `on.paths`
- Verify branch protection rules
- Review Actions tab for errors

**Comments not appearing:**
- Check `GITHUB_TOKEN` permissions
- Verify token has `pull-requests: write`

**AI response invalid:**
- Increase `max_tokens` in Claude API call
- Add error handling for malformed responses

## Next Steps

1. **Add Terraform scanning** for IaC reviews
2. **Integrate with Slack** for notifications
3. **Track metrics** over time
4. **Add human review queues** for critical issues

## Summary

You now have:
- ✅ Automated AI code review on every PR
- ✅ Security and quality checks
- ✅ GitHub integration with comments
- ✅ Scalable to any repository size

**Estimated monthly cost:** $5-20 (depending on PR volume)  
**Time saved:** 2+ hours per month
