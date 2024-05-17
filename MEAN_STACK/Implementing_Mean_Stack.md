# MEAN STACK

The MEAN stack is a popular technology stack used for building dynamic web applications. It's an acronym that stands for MongoDB, Express.js, AngularJS (or Angular), and Node.js. Each component of the stack serves a specific purpose:

1. **MongoDB**: MongoDB is a NoSQL database that stores data in a flexible, JSON-like format. It's known for its scalability and flexibility, making it suitable for handling large volumes of structured or unstructured data.

2. **Express.js**: Express.js is a minimalist web application framework for Node.js. It provides a set of features for building web and mobile applications, including routing, middleware support, and template engines.

3. **Angular (or AngularJS)**: Angular is a front-end JavaScript framework maintained by Google. It's used for building dynamic single-page applications (SPAs) with a modular and structured approach. In the context of the MEAN stack, AngularJS or Angular is responsible for handling the client-side logic and user interface.

4. **Node.js**: Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. It allows developers to run JavaScript code on the server-side, enabling the development of scalable and high-performance web applications. In the MEAN stack, Node.js serves as the server-side runtime environment for running Express.js and handling server-side logic.

Together, these technologies form a full-stack solution for developing modern web applications, where MongoDB handles data storage, Express.js manages server-side logic and routing, Angular (or AngularJS) handles client-side interactions and user interfaces, and Node.js provides the runtime environment for running JavaScript code on the server. This stack is known for its simplicity, flexibility, and ability to handle real-time applications efficiently.


**MEAN STACK IMPLEMENTATION**

PREREQUISITES

1. Have a running EC2 instance on AWS.
2. This time we will connect to our EC2 instance using the AWS CLI

<img width="897" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/fcce7637-096a-4f8c-a96b-ac50ca56b0c2">


## Step 1: Install NodeJS

*Update and Upgrade ubuntu*

```
sudo apt update && sudo apt upgrade -y
```

*Add certificates*
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

<img width="818" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b078856d-b694-4e71-8c50-8890f96e124d">


*Install NodeJS*
```
sudo apt-get install -y nodejs
```

<img width="819" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dda64244-9fa7-4015-a149-34b932904783">


## Step 2: Install MongoDB
For this application, Book records were added to MongoDB that contain book name, isbn number, author, and number of pages.

1. *Download the MongoDB public GPG key*
```
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
```
2. *Add the MongoDB repository*
```
echo "deb [ signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

Mongo public key

3. *Install MongoDB*
```
sudo apt-get install -y mongodb-org
```
<img width="919" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/42e49612-b517-4137-af66-ca22d8c5f890">

4. *Start the Server and Verify that the server is up and running*
```
sudo systemctl status mongod
```
<img width="918" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2eaaf1a8-bb06-4c05-b1d4-37e99f7d7524">

5. *Install body-parser package*

We will install body-parser package which is needed to help process JSON files passed in requests to the server.

```
sudo npm install body-parser
```
6. *Create the project root folder named ‘Books’*

```
mkdir Books && cd Books
```

Initialize the root folder
```
npm init
```

<img width="474" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/fcac9782-e61d-4b4c-a6d0-9dcb52a4561f">

*Add file 'server.js' to Books folder*

```
vi server.js
```

Copy and paste the web server code below into the server.js file.

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose'); // Make sure mongoose is installed and required
const path = require('path'); // To handle static file serving
const app = express();

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

// Middleware
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));

// Routes
require('./apps/routes')(app);

// Start the server
app.set('port', 3300);
app.listen(app.get('port'), () => {
  console.log('Server up: http://localhost:' + app.get('port'));
});
```

## Step 3: Install Express and set up routes to the server
We'll install Express to manage the transfer of book information to and from our MongoDB database. To structure our application data, we'll rely on the Mongoose package, which provides a straightforward schema-based solution. With Mongoose, we'll define a schema that fits our book registry, ensuring that our data is stored in an organized manner within the database.

1. Install express and mongoose

```
sudo npm install express mongoose
```

2. In Books folder we will create a folder named ‘apps’

```
mkdir apps && cd apps
```

In apps folder create a file named routes.js

```
vi routes.js
```
Copy and paste the code below into routes.js

```
const Book = require('./models/book');
const path = require('path');

module.exports = function(app) {
  // Get all books
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find({});
      res.json(books);
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Add a new book
  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const result = await book.save();
      res.json({
        message: "Successfully added book",
        book: result
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Update a book
  app.put('/book/:isbn', async (req, res) => {
    try {
      const updatedBook = await Book.findOneAndUpdate(
        { isbn: req.params.isbn },
        req.body,
        { new: true }
      );
      if (!updatedBook) {
        return res.status(404).json({ error: 'Book not found' });
      }
      res.json({
        message: "Successfully updated the book",
        book: updatedBook
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Delete a book
  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndRemove({ isbn: req.params.isbn });
      if (!result) {
        return res.status(404).json({ error: 'Book not found' });
      }
      res.json({
        message: "Successfully deleted the book",
        book: result
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Serve static files
  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
  });
};
```

3. In the apps folder create a folder named models

```
mkdir models && cd models
```

4. In models create a file named book.js

```
vi book.js
```


﻿```
var mongoose = require('mongoose');
var dbHost =
'mongodb://localhost:27017/test';
mongoose.connect (dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema
name: String,
=
mongoose.Schema( {
isbn: {type: String, index: true},
author: String,
pages: Number
});
var Book =
mongoose.model ('Book', bookSchema);
module.exports
=
mongoose.model ('Book', bookSchema);
```
