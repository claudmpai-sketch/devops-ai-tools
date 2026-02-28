# Node.js Express API Development Guide

Build powerful REST APIs with Node.js and Express. This comprehensive guide covers everything from basic setup to advanced patterns.

## Getting Started

### Installation

```bash
# Create project directory
mkdir my-api
cd my-api

# Initialize npm
npm init -y

# Install Express
npm install express

# Install development dependencies
npm install --save-dev nodemon
```

### Basic Server

Create `server.js`:

```javascript
const express = require('express');
const app = express();

// Parse JSON bodies
app.use(express.json());

// Simple route
app.get('/', (req, res) => {
  res.json({ message: 'Hello World!' });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Run with nodemon (auto-restart on changes):
```bash
npm run dev
```

Add to `package.json`:
```json
"scripts": {
  "dev": "nodemon server.js",
  "start": "node server.js"
}
```

## API Routes

### RESTful Routes

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// In-memory data store
let users = [
  { id: 1, name: 'Martin', email: 'martin@example.com' },
  { id: 2, name: 'Alex', email: 'alex@example.com' }
];

let nextId = 3;

// GET - List all users
app.get('/api/users', (req, res) => {
  res.json(users);
});

// GET - Get user by ID
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json(user);
});

// POST - Create new user
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }
  
  const newUser = {
    id: nextId++,
    name,
    email,
    createdAt: new Date().toISOString()
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT - Update user
app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  const { name, email } = req.body;
  
  if (name) user.name = name;
  if (email) user.email = email;
  
  res.json(user);
});

// DELETE - Delete user
app.delete('/api/users/:id', (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  const [deletedUser] = users.splice(userIndex, 1);
  res.json({ message: 'User deleted', user: deletedUser });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Route Files (Organized Structure)

```
api/
├── server.js
├── routes/
│   ├── index.js
│   ├── users.js
│   └── products.js
├── controllers/
│   ├── userController.js
│   └── productController.js
└── middleware/
    └── errorMiddleware.js
```

### Users Routes

`routes/users.js`:
```javascript
const express = require('express');
const router = express.Router();
const User = require('../models/User');

// GET /api/users
router.get('/', async (req, res) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// GET /api/users/:id
router.get('/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// POST /api/users
router.post('/', async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// PUT /api/users/:id
router.put('/:id', async (req, res) => {
  try {
    const user = await User.update(req.params.id, req.body);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// DELETE /api/users/:id
router.delete('/:id', async (req, res) => {
  try {
    await User.delete(req.params.id);
    res.json({ message: 'User deleted' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

module.exports = router;
```

## Middleware

### Custom Middleware

```javascript
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

module.exports = limiter;
```

### Logging Middleware

`middleware/logger.js`:
```javascript
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.path} - ${Date.now()}`);
  next();
};

module.exports = logger;
```

### Error Handling

`middleware/errorMiddleware.js`:
```javascript
const errorHandler = (err, req, res, next) => {
  console.error(err.stack);
  
  res.status(err.status || 500).json({
    error: err.message || 'Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};

module.exports = errorHandler;
```

## Database Integration

### MongoDB with Mongoose

```bash
npm install mongoose
```

`models/User.js`:
```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Invalid email format']
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: 6
  },
  profile: {
    bio: String,
    avatar: String
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Virtual for full name
userSchema.virtual('fullName').get(function() {
  return this.name;
});

// Pre-save hook (hash password)
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  // Hash password before saving
  const bcrypt = require('bcrypt');
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  
  next();
});

module.exports = mongoose.model('User', userSchema);
```

### SQLite with Better-SQlite

```bash
npm install better-sqlite3
```

```javascript
const Database = require('better-sqlite3');
const db = new Database('database.sqlite');

// Create table
db.prepare(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  )
`).run();

// Insert
const insert = db.prepare('INSERT INTO users (name, email, password) VALUES (?, ?, ?)');
insert.run('Martin', 'martin@example.com', 'hashed_password');

// Select all
const users = db.prepare('SELECT * FROM users').all();

// Select by ID
const user = db.prepare('SELECT * FROM users WHERE id = ?').get(1);
```

## Authentication

### JWT Authentication

```bash
npm install jsonwebtoken bcryptjs
```

`middleware/auth.js`:
```javascript
const jwt = require('jsonwebtoken');

const auth = (req, res, next) => {
  try {
    const token = req.header('Authorization')?.replace('Bearer ', '');
    
    if (!token) {
      return res.status(401).json({ error: 'No authentication token' });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

module.exports = auth;
```

### Login Route

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const User = require('../models/User');

// POST /api/auth/login
app.post('/api/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const token = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    res.json({
      token,
      user: {
        id: user.id,
        email: user.email,
        name: user.name
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## File Uploads

### Multer for File Uploads

```bash
npm install multer
```

`middleware/upload.js`:
```javascript
const multer = require('multer');
const path = require('path');

// Storage configuration
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads/');
  },
  filename: function (req, file, cb) {
    const uniqueName = Date.now() + '-' + Math.round(Math.random() * 1E9) + path.extname(file.originalname);
    cb(null, uniqueName);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpeg|jpg|png|gif/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);
  
  if (mimetype && extname) {
    return cb(null, true);
  }
  cb(new Error('Only images are allowed'));
};

const upload = multer({
  storage: storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB limit
  fileFilter: fileFilter
});

module.exports = upload;
```

### Usage

```javascript
app.post('/api/users/avatar', upload.single('avatar'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  // Save file path to database
  // ...
  
  res.json({
    message: 'Avatar uploaded successfully',
    url: `/uploads/${req.file.filename}`
  });
});
```

## API Documentation

### Swagger/OpenAPI

```bash
npm install swagger-jsdoc swagger-ui-express
```

`schemas/swagger.js`:
```javascript
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'RESTful API documentation'
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      }
    ]
  },
  apis: ['./routes/*.js']
};

const swaggerSpec = swaggerJsdoc(options);

module.exports = swaggerSpec;
```

### API Documentation Routes

`routes/docs.js`:
```javascript
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerSpec = require('../schemas/swagger');

const router = express.Router();

// Serve Swagger UI
router.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

module.exports = router;
```

## Environment Variables

### .env Configuration

```bash
npm install dotenv
```

`.env`:
```env
PORT=3000
NODE_ENV=development
JWT_SECRET=some_super_secret_key
DATABASE_URL=mongodb://localhost:27017/myapi
```

`server.js`:
```javascript
require('dotenv').config();

const app = express();

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

*Generated by AI • Updated February 2026*