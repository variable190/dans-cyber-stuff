# Node/Express Code Review Basics

## 1. Open the project

```bash
code ./project-folder
```

## 2. Find and review the entry point

Look for `app.js`, `server.js` or `index.js` (entry point).

Example entry point:

```js
// ===== Requirements, initialisation and port =====

// Import required libraries
const express = require('express');                         
const cors = require('cors');

// Import route files
const authRoutes = require('./routes/auth-routes');         
const serviceRoutes = require('./routes/service-routes');

// Initialise express object
const app = express();

// Define port
const port = 5000;
// =========================================

// ===== Global middleware goes here, these run on every request before routes =====

// Parses the raw HTTP request body JSON string into a JavaScript object stored in req.body
app.use(express.json());    

// Handles Cross-Origin requests and adds specified HTTP response headers
app.use(cors({                  
    origin: 'https://frontend.example.com',
    methods: ['GET', 'POST'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));     
// =================================================================================

// ===== Mounted routes =====

// Mount the imported routes under their path prefix
app.use("/api/auth", authRoutes);
app.use("/api/service", serviceRoutes);
// ===========================

// ===== Start server =====

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
// ========================
```


## 3. Trace each endpoint

Note:
- Any handling of `req.body`, `req.query`, `req.headers`, `req.params`.
- Any auth checks / middleware.
- What the controller functions actually do.

To trace to the route file CTRL/CMD + click on either:
- `authRoutes` (the variable)
- `'./routes/auth-routes'` (the path string)

### Example route file (`routes/auth-routes.js`)

```js
// ===== Requirements =====

// Import required libraries
const express = require('express');
const router = express.Router();

// Import getUserToken function from '../controllers/auth-controller'
const { getUserToken } = require('../controllers/auth-controller');
// ========================

// ===== Define endpoints =====

// This router is mounted at /api/auth in the entry point
// so this becomes: POST /api/auth/authenticate
// When a matching request arrives, Express calls getUserToken(req, res)
router.post("/authenticate", getUserToken);
// ============================

// ===== export router =====
module.exports = router;
// =========================
```

To trace to the controller file CTRL/CMD + click on either:
- `getUserToken` (the variable)
- `'../controllers/auth-controller'` (the path string)

### Example controller (`controllers/auth-controller.js`)

```js
// ===== controller function =====

function getUserToken(req, res) {

    // User input from parsed JSON body
    const { email } = req.body;

    // Business logic here (validate email, create token, etc.)
    const token = Buffer.from(email).toString('base64');

    // Set body and send JSON response (status defaults to 200)
    res.json({ token: token });
}
// ===============================

// ===== export function =====
module.exports = { getUserToken };
// ===========================
```

After tracing each endpoint review any functions that handle user input.
If required use AI to help understand the code.

We can CTRL/CMD + click on functions to see where they are used in the code.

## 4. Shortlist interesting code

In a massive codebase it may not be possible to go through every function, in which case we can search for interesting code.

### Some dangerous patterns

```js
eval(userInput);                  // code execution
exec(userInput);                  // command injection
Function(userInput)();            // also code execution
JSON.parse(untrustedData);        // potential prototype pollution / deserialization
fs.readFile(userInput);           // path traversal
db.query(`SELECT * FROM users WHERE id = ${userInput}`);  // SQL injection
```

### Search the codebase

In VS Code use CTRL/CMD + SHIFT + F and search for:

| Pattern | Why it matters |
|---------|----------------|
| `eval(` | Code execution |
| `Function(` | Code execution |
| `exec(`/`execSync(`/`spawn(` | Command injection |
| `child_process` | Command execution |
| `JSON.parse(` | Prototype pollution/unsafe deserialisation |
| `fs.readFile`/`fs.writeFile`/`path.join` | Path traversal |
| `` `SELECT ``/`` `INSERT ``/`` `UPDATE ``/`` `DELETE `` | SQL via template literals |
| `.query(`/`.raw(` | Common DB query helpers |
| `jwt.verify`/`jwt.sign` | Auth logic to review |
| `password`/`secret`/`apiKey` | Hardcoded secrets |

### For each hit

1. CTRL/CMD + click into the function
2. Check whether user input reaches it (`req.body`, `req.query`, `req.params`, `req.headers`)
3. If yes; shortlist it for local testing


