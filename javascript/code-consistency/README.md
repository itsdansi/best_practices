# JavaScript Code Consistency Guide

## Table of Contents

1. [Naming Conventions](#naming-conventions)
2. [Variable Declarations](#variable-declarations)
3. [Function Standards](#function-standards)
4. [String Handling](#string-handling)
5. [Object and Array Handling](#object-and-array-handling)
6. [Code Readability](#code-readability-and-maintainability)
7. [Error Handling](#error-handling)
8. [Control Flow](#control-flow-and-logical-operators)
9. [Asynchronous Code](#asynchronous-code)

## Naming Conventions

Maintain consistent naming patterns throughout your codebase:

```javascript
// Constants (UPPER_SNAKE_CASE)
const API_ENDPOINT = "https://api.example.com";
const MAX_RETRY_ATTEMPTS = 3;

// Variables and functions (camelCase)
let currentPageIndex = 0;
function formatDisplayName() {...}

// Classes (PascalCase)
class UserAccount {...}

// Avoid
const temp_value = "";  // Mixed case
function BadlyNamed_Function() {...}
```

## Variable Declarations

Prefer modern block-scoped declarations:

```javascript
// Good - constant reference
const config = { theme: "dark" };

// Good - mutable variable
let attemptCount = 0;

// Avoid - function-scoped
var oldValue = "";

// Proper scoping example
if (shouldProcess) {
  const batchSize = 100; // Block-scoped
  // ...
}
```

## Function Standards

Keep functions short and focused (single responsibility).

- Give functions names that tell you what they do.
- Single responsibility principle.
- Use arrow function for simple logic.

```js
// Good Practice
const add = (a, b) => a + b;
function getUserProfile(userId) { return fetch(`/api/users/${userId}`); }

// Bad Practice
function processDataAndSendEmail() { ... } // Too many responsibilities


// Good - focused function
function calculateOrderTotal(items) {
  return items.reduce((total, item) => total + item.price, 0);
}

// Bad - does too much
function processUserAndSendEmail(user) {
  validateUser(user);
  saveToDatabase(user);
  sendWelcomeEmail(user);
  logAnalytics(user);
}
```

**Default parameters:**

Default function parameters allow named parameters to be initialized with default values if no value or undefined is passed.

```js
// Single Parameter
function greet(name = "Guest") {
  console.log(`Hello, ${name}!`);
}
greet(); // output: Hello, Guest!
greet("John"); // output: Hello, John!

// Multiple Parameters
function multiply(a, b = 1) {
  return a * b;
}
multiply(5); // output: 5
multiply(5, 2); // output: 10
```

## String Handling

Improve readability and reduce errors. Use template literals instead of concatenation.

```js
// Good
const welcomeMessage = `Hello, ${user.name}!
Your last login was ${formatDate(user.lastLogin)}.`;

// Bad
const welcomeMessage =
  "Hello, " +
  user.name +
  "!\n" +
  "Your last login was " +
  formatDate(user.lastLogin) +
  ".";
```

**Be careful with Automatic type conversion**

Beware of numbers, can be converted to `string` or `NaN` by accident. Therefore, consider implementing type testing before sensitive operations or consider using TypeScript for safe typing.

```js
let sum = "5" + 1; // "51" (string concatenation)
// In the code above, typeof sum is a string

let sub = "5" - 1; // 4 (string converted to number)
// In the code obove, typeof sub in a number

let lang = "JavaScript"; // typeof name is string
lang = 15; // changes typeof x to a number
```

## Object and Array Handling

Use modern ES6+ features. Use Object / Array destructuring, it allows you to extract multiple properties from an object or elements from an array in a single statement, reducing the amount of code you need to write.

```js
// Good Practice
const user = { name: "Alice", age: 25 };
const { name, age } = user;

const numbers = [1, 2, 3];
const [first, second] = numbers;

const { height = 180 } = person; // uses default value if height is undefined

// Bad Practice
const name = user.name;
const age = user.age;
```

Use `map`, `filter`, `reduce` array methods instead of `for` loops for cleaner code.

```js
// Good Practice
const numbers = [1, 2, 3, 4, 5];
const squared = numbers.map((num) => num * num);

// Bad Practice
const squared = [];
for (let i = 0; i < numbers.length; i++) {
  squared.push(numbers[i] * numbers[i]);
}
```

## Code Readability and Maintainability

- Use consistent formatting.
- Indenting and spaces: 2 spaces (To be configured in text editor of choice)
- Linting:
  - Eslint: code recommendation with formatting off
  - prettier: for code formatting
- Comments:
  - Comments should be available for things that are complicated.
  - Add comments only where necessary.

## Error Handling

Prevent app crashes due to unhandled errors. Use `try/catch` to catch errors in risky spots.

```js
try {
  riskyCode();
} catch (error) {
  console.log(error, contextDetails);
  showFallback();
}
```

## Control Flow and Logical Operators

Use `===` instead of `==` to avoid type coercion.

```js
// Good Practice
console.log(5 === "5"); // false

// Bad Practice
console.log(5 == "5"); // true (unexpected behavior)
```

Use Optional Chaining (`?.`) to avoid errors.

```js
// Good Practice
console.log(user.profile?.name); // Avoids TypeError

// Bad Practice
console.log(user.profile.name); // Error if profile is undefined
```

```js
//Preffred way
console.log(userA ?? userB); // onlyif userA is either null or undefined it choose userB

//Okay not bad
console.log(userA ? userA : userB);
```

## Asynchronous Code

Asynchronous JavaScript is essential for handling tasks like API calls, file operations, or database queries efficiently. Below are the best practices for writing clean and maintainable asynchronous code.

**Use `async/await` Instead of Callbacks:**

```js
async function fetchUserData(userId) {
  try {
    const response = await fetch(`https://api.example.com/users/${userId}`);
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Error fetching user data:", error);
  }
}
```

**Handle Errors with Try-Catch:**

Use `try...catch` inside async functions

```js
async function getData() {
  try {
    const response = await fetch("https://api.example.com/data");
    if (!response.ok) throw new Error("Failed to fetch data");
    return await response.json();
  } catch (error) {
    console.error("Error:", error);
  }
}
```

**Parallel Execution:**

Use `Promise.all` for running multiple independent async tasks concurrently.

```js
async function fetchMultipleUsers(userIds) {
  const userPromises = userIds.map((id) =>
    fetch(`https://api.example.com/users/${id}`).then((res) => res.json())
  );
  return await Promise.all(userPromises);
}
```

Use `Promise.allSettle` Similar to Promise.all(), but it waits for all promises to settle (either resolved or rejected) and returns their results.

```js
async function fetchUsers(userIds) {
  const userPromises = userIds.map((id) =>
    fetch(`https://api.example.com/users/${id}`).then((res) => res.json())
  );
  const results = await Promise.allSettled(userPromises);

  return results.map((result) =>
    result.status === "fulfilled" ? result.value : { error: "Failed to fetch" }
  );
}
```

Use `Promise.race` Returns the result of the first promise that settles (either resolved or rejected).
Use cases: Fetching from multiple servers and using the fastest

```js
async function fetchFastestUser(userId) {
  const servers = [
    `https://api1.example.com/users/${userId}`,
    `https://api2.example.com/users/${userId}`,
  ];

  const userPromises = servers.map((url) =>
    fetch(url).then((res) => res.json())
  );
  return await Promise.race(userPromises);
}
```

**Execute Cleanup Code:**

Use `finally` to execute cleanup code always.

```js
async function processData() {
  try {
    console.log("Fetching data...");
    const data = await fetch("https://api.example.com/data").then((res) =>
      res.json()
    );
    console.log("Data received:", data);
  } catch (error) {
    console.error("Error fetching data:", error);
  } finally {
    console.log("Cleanup: Closing connections...");
  }
}
```

**Avoid Mixing `async/await` and `.then/.catch`**

Stick to only one approach.

```js
// Good Practice
async function getData() {
  try {
    const response = await fetch("https://api.example.com/data");
    return await response.json();
  } catch (error) {
    console.error("Error fetching data:", error);
  }
}

// Bad Practice
async function getData() {
  return fetch("https://api.example.com/data")
    .then((response) => response.json())
    .catch((error) => console.error("Error fetching data:", error));
}
```
