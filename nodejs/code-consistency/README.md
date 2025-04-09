# NodeJs Code Consistency Guide

## Table of Contents

1. [Error Handling Middleware](#error-handeling-middleware)
2. [Environment Configuration](#environment-configuration)
3. [Async/Await Pattern](#async-pattern)
4. [Logging Strategy](#logging-strategy)
5. [Request Validation](#request-validation)
6. [Use Security Headers](#user-security-headers)
7. [Use Rate Limiter](#use-rate-limiter)
8. [Proper Shutdown](#proper-shutdown)
9. [File System Operations](#file-system-operations)
10. [Use Database Transactions](#use-database-transactions)

## Error Handling Middleware

Always implement proper error handling in Express applications:

```javascript
// Good practice
app.use(async (err, req, res, next) => {
  await logError(err);
  res.status(500).json({
    error:
      process.env.NODE_ENV === "production" ? "Something failed" : err.message,
  });
});

// Bad - No error handling
app.get("/user", async (req, res) => {
  const user = await getUser(req.params.id);
  res.json(user); // Crashes if getUser fails
});
```

## Environment Configuration

Use environment variables properly:

```javascript
// Good practice
// config.js
require("dotenv").config();

module.exports = {
  PORT: process.env.PORT || 3000,
  DB_URI: process.env.DB_URI || "mongodb://localhost/dev",
  JWT_SECRET: process.env.JWT_SECRET || "unsafe_dev_secret",
};

// Bad - Hardcoded values
const DB_URI = "mongodb://prod-db:27017"; // Committed to repo
```

## Async/Await Pattern

Proper async/await usage in routes:

```javascript
// Good practice
router.get("/posts", async (req, res, next) => {
  try {
    const posts = await Post.find().lean();
    res.json(posts);
  } catch (err) {
    next(err); // Pass to error handler
  }
});

// Bad - Unhandled promise rejection
router.get("/posts", (req, res) => {
  Post.find().then((posts) => res.json(posts));
});
```

## Logging Strategy

Implement structured logging:

```javascript
// Good practice
// winston-config.js
const winston = require("winston");

module.exports = winston.createLogger({
  level: "info",
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: "error.log", level: "error" }),
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      ),
    }),
  ],
});

// Bad - Console.log everywhere
console.log("User logged in"); // No timestamps, levels, or structure
```

## Request Validation

Validate incoming requests:

```javascript
// Using Joi for validation
const Joi = require("joi");

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
});

router.post("/register", async (req, res) => {
  try {
    await userSchema.validateAsync(req.body);
    // Proceed with registration
  } catch (err) {
    res.status(400).json({ error: err.details[0].message });
  }
});

// Bad - No validation
router.post("/register", (req, res) => {
  createUser(req.body); // Accepts any input
});
```

## Use Security Headers

Security Headers are crucial in any applicatio. We can manually include http headers or use a dedicated package called helmet.

```javascript
// security.js
const helmet = require("helmet");

module.exports = (app) => {
  app.use(helmet());
};

// Bad - No security headers
const app = express(); // Vulnerable to common attacks
```

## Use Rate Limiter

Using rate-limiter or throttler are so crucial these days to prevent unintented D-DOS attact or performance impact.

```javascript
// security.js
const rateLimit = require("express-rate-limit");

app.use(
  rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per window
  })
);

// Bad - No security headers
const app = express(); // Vulnerable to common attacks
```

## Proper Shutdown

Graceful shutdown handling:

```javascript
// server.js
process.on("SIGTERM", () => {
  console.log("SIGTERM received. Shutting down gracefully");
  server.close(() => {
    db.disconnect();
    process.exit(0);
  });
});

// Bad - No shutdown handling
const server = app.listen(3000); // Kills connections immediately
```

## File System Operations

Instead of reading entire file contents into memory at once using fs.readFile() and risking memory issues with large files,
use Node.js streams to process large files by piping a read stream through gzip compression to a write stream.

```javascript
// Good -Using streams for large files
const fs = require("fs");
const zlib = require("zlib");

function processLargeFile(inputPath, outputPath) {
  return fs
    .createReadStream(inputPath)
    .pipe(zlib.createGzip())
    .pipe(fs.createWriteStream(outputPath));
}

// Bad - Loading entire file to memory
fs.readFile("huge-file.txt", (err, data) => {
  // 2GB file crashes the process
});
```

## Use Database Transactions

Use database transactions with session management to ensure atomic operations, including commit/rollback and error handling.

```javascript
// Good - Using proper transaction handling
const session = await mongoose.startSession();
session.startTransaction();

try {
  const order = await Order.create([orderData], { session });
  await Inventory.updateOne(
    { product: order.product },
    { $inc: { quantity: -order.quantity } },
    { session }
  );
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}

// Bad - No transaction handling
await Order.create(orderData);
await Inventory.updateOne(
  { product: orderData.prod// Bad
uct },
  { $inc: { quantity: -orderData.quantity } }
);
// Data becomes inconsistent if one operation fails
```
