# Python Automation Scripts: Real-World Examples

Automation in Python can save you hours of manual work. This guide shows you 10 practical automation scripts for everyday tasks.

## 1. File Organization Script

Automatically sort files into folders by extension:

```python
import os
import shutil
from pathlib import Path

def organize_folder(folder_path):
    """Sort files by extension"""
    folder = Path(folder_path)
    
    for file in folder.iterdir():
        if file.is_file():
            ext = file.suffix[1:].lower() or 'no_extension'
            ext_folder = folder / ext
            ext_folder.mkdir(exist_ok=True)
            file.rename(ext_folder / file.name)
            print(f"Moved {file.name} to {ext}/")

# Usage
organize_folder('Downloads')
```

## 2. Web Scraping Script

Extract data from websites using BeautifulSoup:

```python
import requests
from bs4 import BeautifulSoup
import csv

def scrape_prices(url):
    """Scrape product prices from e-commerce site"""
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    products = []
    for product in soup.find_all('div', class_='product'):
        name = product.find('h2').text.strip()
        price = product.find('span', class_='price').text.strip()
        products.append({'name': name, 'price': price})
    
    # Save to CSV
    with open('products.csv', 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=['name', 'price'])
        writer.writeheader()
        writer.writerows(products)
    
    return products

# Usage
products = scrape_prices('https://example.com/products')
print(f"Found {len(products)} products")
```

## 3. Email Automation

Send bulk emails with personalized content:

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import csv

def send_bulk_emails(emails_file, subject, body_template):
    """Send personalized emails to recipients"""
    
    # Connect to email server
    server = smtplib.SMTP('smtp.gmail.com', 587)
    server.starttls()
    server.login('your_email@gmail.com', 'your_password')
    
    with open(emails_file, 'r') as f:
        reader = csv.DictReader(f)
        
        for row in reader:
            msg = MIMEMultipart()
            msg['From'] = 'your_email@gmail.com'
            msg['To'] = row['email']
            msg['Subject'] = subject
            
            custom_body = body_template.format(name=row['name'])
            msg.attach(MIMEText(custom_body, 'html'))
            
            server.send_message(msg)
            print(f"Sent to {row['email']}")
    
    server.quit()

# Usage
send_bulk_emails(
    'email_list.csv',
    'Check out our new products!',
    '<p>Hi {name},</p>
     <p>We have new products that might interest you!</p>'
)
```

## 4. Backup Automation

Automatically backup important files:

```python
import os
import shutil
from datetime import datetime

def backup_importante_files(source_dir, backup_dir):
    """Backup files with timestamps"""
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_name = f"backup_{timestamp}"
    backup_path = os.path.join(backup_dir, backup_name)
    
    # Create backup directory
    os.makedirs(backup_path, exist_ok=True)
    
    # Copy files
    shutil.copytree(source_dir, backup_path)
    
    # Create metadata file
    with open(os.path.join(backup_path, 'backup_info.txt'), 'w') as f:
        f.write(f"Backup created: {timestamp}\n")
        f.write(f"Source: {source_dir}\n")
        f.write(f"Files backed up: {len(os.listdir(source_dir))}")
    
    print(f"Backup completed: {backup_path}")

# Usage
backup_importante_files(
    'C:/Documents',
    'D:/Backups'
)
```

## 5. PDF Processing

Combine, split, or modify PDF files:

```python
from PyPDF2 import PdfMerger, PdfReader, PdfWriter

def merge_pdfs(input_files, output_file):
    """Combine multiple PDFs into one"""
    merger = PdfMerger()
    
    for pdf_file in input_files:
        merger.append(pdf_file)
    
    merger.write(output_file)
    merger.close()
    print(f"Merged {len(input_files)} PDFs into {output_file}")

def split_pdf(input_pdf, output_dir):
    """Split PDF into individual pages"""
    reader = PdfReader(input_pdf)
    
    for i, page in enumerate(reader.pages):
        writer = PdfWriter()
        writer.add_page(page)
        
        output_file = os.path.join(output_dir, f'page_{i+1}.pdf')
        with open(output_file, 'wb') as f:
            writer.write(f)
    
    print(f"Split {input_pdf} into {len(reader.pages)} pages")

# Usage
merge_pdfs(['file1.pdf', 'file2.pdf'], 'combined.pdf')
split_pdf('document.pdf', 'output/')
```

## 6. Schedule Tasks

Automate scripts to run at specific times:

```python
from schedule import run_pending
import time

def daily_task():
    """Run every day at 6 AM"""
    print("Running daily backup...")
    backup_importante_files('C:/Documents', 'D:/Backups')

def weekly_task():
    """Run every Sunday at 10 AM"""
    print("Running weekly cleanup...")
    # Your cleanup logic here
    pass

def monthly_task():
    """Run first of every month at 9 AM"""
    print("Running monthly report...")
    # Your report logic here
    pass

# Schedule tasks
schedule.every().day.at("06:00").do(daily_task)
schedule.every().sunday.at("10:00").do(weekly_task)
schedule.every().1st_monday.at("09:00").do(monthly_task)

# Run scheduler
while True:
    run_pending()
    time.sleep(60)
```

## 7. Social Media Automation

Post content to social media platforms:

```python
import tweepy
from datetime import datetime

class SocialMediaBot:
    def __init__(self, api_key, api_secret, access_token, access_secret):
        client = tweepy.Client(
            consumer_key=api_key,
            consumer_secret=api_secret,
            access_token=access_token,
            access_token_secret=access_secret
        )
        self.client = client
    
    def post_tweet(self, text):
        """Post a tweet"""
        try:
            self.client.create_tweet(text=text)
            print(f"Tweet posted: {text}")
            return True
        except Exception as e:
            print(f"Error posting tweet: {e}")
            return False
    
    def schedule_tweet(self, text, time_to_schedule):
        """Schedule tweet for later"""
        from datetime import datetime, timedelta
        scheduled_time = datetime.now() + time_to_schedule
        
        # For now, just print (real scheduling uses external tool)
        print(f"Would schedule: {text} at {scheduled_time}")

# Usage
bot = SocialMediaBot('key', 'secret', 'token', 'secret')
bot.post_tweet("Just deployed a new feature! 🚀 #python #devops")
```

## 8. Image Processing

Resize, watermark, or convert images:

```python
from PIL import Image, ImageDraw, ImageFont
import os

def resize_images(input_folder, output_folder, size=(800, 600)):
    """Resize all images to specified dimensions"""
    os.makedirs(output_folder, exist_ok=True)
    
    for file in os.listdir(input_folder):
        if file.lower().endswith(('.png', '.jpg', '.jpeg', '.gif', '.bmp')):
            input_path = os.path.join(input_folder, file)
            output_path = os.path.join(output_folder, file)
            
            img = Image.open(input_path)
            img_resized = img.resize(size)
            img_resized.save(output_path)
            print(f"Resized {file}")

def add_watermark(input_image, output_image, text="© YourName"):
    """Add watermark to image"""
    image = Image.open(input_image)
    draw = ImageDraw.Draw(image)
    
    # Try to use font, fallback to default
    try:
        font = ImageFont.truetype("arial.ttf", 20)
    except:
        font = ImageFont.load_default()
    
    # Calculate text position
    bbox = draw.textbbox((0, 0), text, font=font)
    text_width = bbox[2] - bbox[0]
    text_height = bbox[3] - bbox[1]
    
    position = (image.width - text_width - 10, image.height - text_height - 10)
    draw.text(position, text, font=font, fill=(255, 255, 255, 128))
    
    image.save(output_image)
    print(f"Watermarked {input_image}")

# Usage
resize_images('images/', 'images_resized/', size=(600, 400))
add_watermark('photo.jpg', 'photo_watermarked.jpg')
```

## 9. Web Browser Automation

Control web browser with Selenium:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

def automate_web_task(url, tasks):
    """Automate browser tasks"""
    options = webdriver.ChromeOptions()
    options.add_argument('--headless')  # Run without GUI
    driver = webdriver.Chrome(service=Service(), options=options)
    
    try:
        driver.get(url)
        wait = WebDriverWait(driver, 10)
        
        for task in tasks:
            action_type = task['type']
            target = task['target']
            value = task.get('value')
            
            if action_type == 'click':
                element = wait.until(EC.element_to_be_clickable((By.XPATH, target)))
                element.click()
                print(f"Clicked {target}")
            
            elif action_type == 'fill':
                element = wait.until(EC.presence_of_element_located((By.XPATH, target)))
                element.clear()
                element.send_keys(value)
                print(f"Filled {target}")
            
            elif action_type == 'screenshot':
                driver.save_screenshot(f"screen_{time.time()}.png")
                print("Screenshot saved")
            
            time.sleep(1)  # Wait between actions
        
        return driver
        
    finally:
        driver.quit()

# Usage
tasks = [
    {'type': 'click', 'target': '//button[@id="login"]'},
    {'type': 'fill', 'target': '//input[@name="username"]', 'value': 'your_user'},
    {'type': 'fill', 'target': '//input[@name="password"]', 'value': 'your_pass'},
    {'type': 'click', 'target': '//button[@type="submit"]'},
    {'type': 'screenshot', 'target': None},
]

driver = automate_web_task('https://example.com/login', tasks)
```

## 10. Database Automation

Automate database operations:

```python
import sqlite3
import pandas as pd
from datetime import datetime

def backup_database(db_path, backup_dir):
    """Create database backup"""
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_file = f"{backup_dir}/backup_{timestamp}_db.sqlite"
    
    try:
        conn = sqlite3.connect(db_path)
        backup_conn = sqlite3.connect(backup_file)
        
        # Backup all data
        for table_name in pd.read_sql("SELECT name FROM sqlite_master WHERE type='table';", conn)['name']:
            df = pd.read_sql(f"SELECT * FROM {table_name}", conn)
            df.to_sql(table_name, backup_conn, index=False)
        
        backup_conn.close()
        conn.close()
        print(f"Database backed up to {backup_file}")
        
    except Exception as e:
        print(f"Backup failed: {e}")

def export_tables_to_csv(db_path, export_dir):
    """Export all database tables to CSV"""
    if not os.path.exists(export_dir):
        os.makedirs(export_dir)
    
    conn = sqlite3.connect(db_path)
    
    for table_name in pd.read_sql("SELECT name FROM sqlite_master WHERE type='table';", conn)['name']:
        df = pd.read_sql(f"SELECT * FROM {table_name}", conn)
        csv_file = os.path.join(export_dir, f"{table_name}.csv")
        df.to_csv(csv_file, index=False)
        print(f"Exported {table_name} to {csv_file}")
    
    conn.close()

# Usage
backup_database('myapp.db', 'backups/')
export_tables_to_csv('myapp.db', 'exports/')
```

## Running Automations

### Using Task Scheduler (Windows)

Create a batch file (`run_automation.bat`):
```batch
@echo off
cd "C:\automation_scripts"
python file_organizer.py
python backup_script.py
echo Automation completed at %date% %time%
```

Then schedule it via Windows Task Scheduler:
1. Open `taskschd.msc`
2. Create Basic Task
3. Set trigger (daily, weekly, etc.)
4. Set action to run the batch file

### Using Cron (Linux/Mac)

Add to crontab (`crontab -e`):
```bash
# Run file organizer daily at 6 AM
0 6 * * * /usr/bin/python3 /home/user/scripts/organize_files.py

# Run backup every Sunday at 10 AM
0 10 * * 0 /usr/bin/python3 /home/user/scripts/backup.py

# Run web scraper every hour
0 * * * * /usr/bin/python3 /home/user/scripts/scrape_website.py
```

---

*Generated by AI • Updated February 2026*