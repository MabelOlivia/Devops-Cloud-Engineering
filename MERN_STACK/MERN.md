# MERN STACK

The MERN stack is a popular software stack used for building modern web applications. It's an acronym that stands for:

1. **MongoDB**: A NoSQL database that uses a document-oriented data model. MongoDB is known for its flexibility and scalability, making it suitable for handling large volumes of data.

2. **Express.js**: A web application framework for Node.js. Express.js simplifies the process of building web applications and APIs by providing a set of robust features for routing, middleware, and more.

3. **React**: A JavaScript library for building user interfaces. React is maintained by Facebook and is widely used for creating dynamic and interactive UI components in web applications.

4. **Node.js**: A server-side JavaScript runtime environment. Node.js allows developers to run JavaScript code on the server, enabling them to build scalable and high-performance web applications.

Together, these technologies provide a full-stack JavaScript development environment, allowing developers to build end-to-end web applications using a single programming language (JavaScript) throughout the entire stack. This simplifies the development process and promotes code reusability between the client and server sides of the application.


---

**MERN STACK IMPLEMENTATION**

**PREREQUISITES**

1. Have a running EC2 instance on AWS.
2. Connect to the EC2 instance using SSH from the Windows Terminal.

**BACKEND CONFIGURATION**

Update Ubuntu:

```bash
sudo apt update
```

Upgrade Ubuntu:

```bash
sudo apt upgrade
```

Get the location of Node.js software from Ubuntu repositories:

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

Installing Node.js on the Server:

```bash
sudo apt-get install -y nodejs
```

Verify the installation of Node:

```bash
node -v
```

Verify the installation of npm:

```bash
npm -v
```

**Application Code Set-Up**

Create a new directory for the To-Do project:

```bash
mkdir Todo
```

Change Directory to the new one:

```bash
cd Todo
```

Initialize the project:

```bash
npm init
```

Create a file index.js:

```bash
touch index.js
```

Install dotenv module:

```bash
npm install dotenv
```

Open the index.js file:

```bash
vim index.js
```

Copy and paste the following code into the open file:

```javascript
const express = require('express');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 5000;

app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
    next();
});

app.use((req, res, next) => {
    res.send('Welcome to Express');
});

app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});
```

**INSTALL EXPRESSJS**

Install Express:

```bash
npm install express
```

Start the server:

```bash
node index.js
```

**OPEN PORT IN EC2 SECURITY GROUPS**

Open port 5000 in your EC2 Security Groups.

**ACCESS THE SERVER**

Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:

```http
http://<PublicIP-or-PublicDNS>:5000
```

**CREATING ROUTES**

Create a folder `routes`:

```bash
mkdir routes
```

Change directory to `routes`:

```bash
cd routes
```

Create a file `api.js`:

```bash
touch api.js
```

Open the `api.js` file:

```bash
vim api.js
```

Copy and paste the following commands into `api.js`:

```javascript
const express = require('express');

const router = express.Router();

router.get('/todos', (req, res, next) => {
    // Code for getting all todos
});

router.post('/todos', (req, res, next) => {
    // Code for creating a new todo
});

router.delete('/todos/:id', (req, res, next) => {
    // Code for deleting a todo by id
});

module.exports = router;
```

---

This guide outlines the steps to set up a MERN stack project on an AWS EC2 instance, configure the backend, install Express.js, open ports, and create routes. Let me know if you need further assistance!
