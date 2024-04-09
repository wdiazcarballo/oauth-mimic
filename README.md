# oauth-mimic
an example on how to develop an oAuth-like workflow to practice developing API endpoints.

---

## Lab Handout: Simulating OAuth Workflow

### Objective:
To create a simplified version of the OAuth workflow using a Node.js Express server, which will illustrate how a client application can obtain an access token to make authenticated requests to a protected resource.

### Directory Structure:
```
oauth-simulator/
│
├── package.json
├── app.js
│
├── controllers/
│   ├── authController.js
│
├── models/
│   ├── clientModel.js
│   ├── codeModel.js
│   ├── tokenModel.js
│   ├── userModel.js
│
└── routes/
    ├── authRoutes.js
```

### Initialization and File Content:

**Step 1: Initialize npm and Install Dependencies**
```sh
mkdir oauth-simulator
cd oauth-simulator
npm init -y
npm install express body-parser dotenv
```

This creates the `package.json` file and installs necessary dependencies.

**package.json** (auto-generated with `npm init -y`, dependencies added with `npm install`):
```json
{
  "name": "oauth-simulator",
  "version": "1.0.0",
  "description": "A simple OAuth simulator for educational purposes",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.17.1",
    "body-parser": "^1.19.0",
    "dotenv": "^8.2.0"
  }
}
```

**app.js**:
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const authRoutes = require('./routes/authRoutes');

const app = express();

app.use(bodyParser.json());
app.use('/auth', authRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

**controllers/authController.js**:
```javascript
// Assume these are utility functions and models with appropriate methods.
const { generateClientId, generateAuthCode, exchangeCodeForToken, getUserInfoByToken, refreshToken } = require('../models/tokenModel');
const User = require('../models/userModel');

exports.registerApplication = (req, res) => {
  // ...
};

exports.authorizeApplication = (req, res) => {
  // ...
};

exports.token = (req, res) => {
  // ...
};

exports.userInfo = (req, res) => {
  // ...
};

exports.refreshToken = (req, res) => {
  // ...
};
```

**models/clientModel.js, codeModel.js, tokenModel.js, userModel.js**:
```javascript
// Each model file would export functions that manipulate or retrieve data for that model.
// For example, clientModel.js might look something like this:
class Client {
  constructor(name) {
    this.name = name;
    this.clientId = generateClientId(name);
  }

  // Further methods to handle client data
}

module.exports = Client;
```

**routes/authRoutes.js**:
```javascript
const express = require('express');
const authController = require('../controllers/authController');

const router = express.Router();

router.post('/register-application', authController.registerApplication);
router.get('/authorize', authController.authorizeApplication);
router.post('/token', authController.token);
router.get('/user-info', authController.userInfo);
router.post('/refresh-token', authController.refreshToken);

module.exports = router;
```

### Execution:
Run the server using `npm start` and navigate to the endpoints to simulate each step of the OAuth workflow.

### Conclusion:
This lab will familiarize you with the OAuth protocol, specifically focusing on how clients are registered, how authorization codes are generated and exchanged for access tokens, and how those tokens are used to access protected resources.

**Security Note:**
Students should be made aware that in a real-world scenario, client secrets must be protected, and all exchanges should occur over HTTPS. Additionally, proper validation and error handling must be implemented to prevent security vulnerabilities.

---

Here is a simple example of what the controller and model code could look like when interacting with a database. For this example, I will use an in-memory JavaScript object to simulate database storage. In a production scenario, you would replace these with calls to a real database such as MongoDB or MySQL.

### Controllers

**authController.js:**
```javascript
const { Client, AuthCode, Token, User } = require('../models'); // Assuming index.js is handling model exports

exports.registerApplication = (req, res) => {
  const client = new Client(req.body.applicationName);
  res.json({ client_id: client.clientId });
};

exports.authorizeApplication = (req, res) => {
  const client = Client.findByClientId(req.query.client_id);
  if (client) {
    const user = User.findById(req.session.userId);
    if (user) {
      const authCode = new AuthCode(client.clientId, user.id);
      res.json({ authorization_code: authCode.code });
    } else {
      res.status(401).send('User not authenticated');
    }
  } else {
    res.status(404).send('Client not found');
  }
};

exports.token = (req, res) => {
  const authCode = AuthCode.findByCode(req.body.code);
  if (authCode) {
    const token = new Token(authCode.userId);
    res.json({ access_token: token.accessToken, refresh_token: token.refreshToken });
  } else {
    res.status(400).send('Invalid authorization code');
  }
};

exports.userInfo = (req, res) => {
  const token = Token.findByAccessToken(req.headers.authorization);
  if (token) {
    const user = User.findById(token.userId);
    res.json(user);
  } else {
    res.status(403).send('Invalid token');
  }
};

exports.refreshToken = (req, res) => {
  const token = Token.findByRefreshToken(req.body.refresh_token);
  if (token) {
    token.refreshAccessToken();
    res.json({ access_token: token.accessToken });
  } else {
    res.status(400).send('Invalid refresh token');
  }
};
```

### Models

**ClientModel.js:**
```javascript
const clients = {}; // Simulate database table for clients

class Client {
  constructor(name) {
    this.name = name;
    this.clientId = this.generateClientId();
    clients[this.clientId] = this;
  }

  generateClientId() {
    return 'client_' + Date.now() + Math.random();
  }

  static findByClientId(clientId) {
    return clients[clientId];
  }
}

module.exports = Client;
```

**AuthCodeModel.js:**
```javascript
const authCodes = {}; // Simulate database table for authorization codes

class AuthCode {
  constructor(clientId, userId) {
    this.clientId = clientId;
    this.userId = userId;
    this.code = this.generateAuthCode();
    authCodes[this.code] = this;
  }

  generateAuthCode() {
    return 'code_' + Date.now() + Math.random();
  }

  static findByCode(code) {
    return authCodes[code];
  }
}

module.exports = AuthCode;
```

**TokenModel.js:**
```javascript
const tokens = {}; // Simulate database table for tokens

class Token {
  constructor(userId) {
    this.userId = userId;
    this.accessToken = this.generateAccessToken();
    this.refreshToken = this.generateRefreshToken();
    tokens[this.accessToken] = this;
  }

  generateAccessToken() {
    return 'access_' + Date.now() + Math.random();
  }

  generateRefreshToken() {
    return 'refresh_' + Date.now() + Math.random();
  }

  refreshAccessToken() {
    this.accessToken = this.generateAccessToken();
  }

  static findByAccessToken(accessToken) {
    return tokens[accessToken];
  }

  static findByRefreshToken(refreshToken) {
    return Object.values(tokens).find(token => token.refreshToken === refreshToken);
  }
}

module.exports = Token;
```

**UserModel.js:**
```javascript
const users = {}; // Simulate database table for users

class User {
  constructor(username) {
    this.id = 'user_' + Date.now() + Math.random();
    this.username = username;
    users[this.id] = this;
  }

  static findById(id) {
    return users[id];
  }
}

module.exports = User;
```

**index.js (for models):**
```javascript
const Client = require('./ClientModel');
const AuthCode = require('./AuthCodeModel');
const Token = require('./TokenModel');
const User = require('./UserModel');

module.exports = {
  Client,
  AuthCode,
  Token,
  User
};
```

These models currently utilize an in-memory object store to simulate database interactions, which you would replace with actual database operations using an ORM like Sequelize for SQL databases or Mongoose for MongoDB.

To make these models work in a real application,

 you would need to replace the object store with actual database CRUD operations, handle errors properly, and manage sessions securely.
