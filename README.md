To create a REST API using `json-server` and extend user authentication using `express.js`, follow these steps:

Step 1: Set up the project
Create a new directory for your project and navigate into it using the terminal.

```
mkdir json-server-auth
cd json-server-auth
```

Step 2: Initialize the project and install required dependencies
Initialize the project and install the necessary dependencies (json-server, express, and jsonwebtoken for authentication).

```
npm init -y
npm install json-server express jsonwebtoken
```

Step 3: Create a mock JSON data file
Create a file named `db.json` in the root of your project directory. This file will act as your database for `json-server`.

```json
// db.json
{
  "users": [
    {
      "id": 1,
      "username": "user1",
      "password": "password1"
    },
    {
      "id": 2,
      "username": "user2",
      "password": "password2"
    }
  ]
}
```

Step 4: Create the JSON Server script
In your `package.json`, add the following script to start the `json-server`.

```json
// package.json
"scripts": {
  "start:json-server": "json-server --watch db.json --port 3000"
},
```

Step 5: Create the Express.js authentication server
Create a new file named `server.js` in the root of your project directory for the Express.js server.

```javascript
// server.js
const express = require('express');
const bodyParser = require('body-parser');
const jwt = require('jsonwebtoken');

const app = express();
const port = 4000;
const secretKey = 'your-secret-key'; // Replace with your own secret key

// Middleware to parse JSON data
app.use(bodyParser.json());

// Custom middleware for authentication
function authenticateToken(req, res, next) {
  const token = req.header('Authorization');
  if (!token) {
    return res.status(401).json({ message: 'Unauthorized' });
  }

  jwt.verify(token, secretKey, (err, user) => {
    if (err) {
      return res.status(403).json({ message: 'Forbidden' });
    }

    req.user = user;
    next();
  });
}

// Login endpoint - generate a JWT and return it
app.post('/login', (req, res) => {
  const { username, password } = req.body;

  // Replace this with a proper authentication check, e.g., querying the db.json
  // Here, we are just checking if the username and password match a user in db.json
  const user = require('./db.json').users.find(
    (u) => u.username === username && u.password === password
  );

  if (!user) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  // Create and sign a JWT with the user's information
  const token = jwt.sign(user, secretKey);

  // Return the JWT
  res.json({ token });
});

// Protected endpoint - only accessible with a valid token
app.get('/protected', authenticateToken, (req, res) => {
  // If the token is valid, req.user will contain the user information from the JWT payload
  res.json({ message: 'You have accessed the protected endpoint', user: req.user });
});

// Start the server
app.listen(port, () => {
  console.log(`Authentication server is running on http://localhost:${port}`);
});
```

Step 6: Start the servers
Now, you can start both the JSON Server and the Express.js authentication server.

In one terminal, run the JSON Server:

```
npm start:json-server
```

In another terminal, run the Express.js authentication server:

```
node server.js
```

Step 7: Test the API
You can use tools like `curl` or Postman to test the API.

1. To log in and get a JWT token, send a POST request to `http://localhost:4000/login` with the following JSON body:

```json
{
  "username": "user1",
  "password": "password1"
}
```

2. Copy the token you received from the login response.

3. To access the protected endpoint, send a GET request to `http://localhost:4000/protected` with an `Authorization` header containing the JWT token you copied:

```
Authorization: your-jwt-token
```

If the token is valid, you will get a response with the message "You have accessed the protected endpoint" and the user information extracted from the token payload.

That's it! You now have a REST API using `json-server` for the database and `express.js` for user authentication with JWT. Remember that this is a simplified example for demonstration purposes, and in a real-world scenario, you would need a proper authentication strategy and a secure secret key.
