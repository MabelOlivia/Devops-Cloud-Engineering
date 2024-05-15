# MERN STACK

The MERN stack is a popular software stack used for building modern web applications. It's an acronym that stands for:

1. **MongoDB**: A NoSQL database that uses a document-oriented data model. MongoDB is known for its flexibility and scalability, making it suitable for handling large volumes of data.

2. **Express.js**: A web application framework for Node.js. Express.js simplifies the process of building web applications and APIs by providing a set of robust features for routing, middleware, and more.

3. **React**: A JavaScript library for building user interfaces. React is maintained by Facebook and is widely used for creating dynamic and interactive UI components in web applications.

4. **Node.js**: A server-side JavaScript runtime environment. Node.js allows developers to run JavaScript code on the server, enabling them to build scalable and high-performance web applications.

Together, these technologies provide a full-stack JavaScript development environment, allowing developers to build end-to-end web applications using a single programming language (JavaScript) throughout the entire stack. This simplifies the development process and promotes code reusability between the client and server sides of the application.


---
<img width="932" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/770e32f7-574a-47e8-a32b-fee2da42a5c2">

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
![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 001](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c4ca4207-7e57-4693-9427-5ba9c77fb3a9)


Verify the installation of npm:

```bash
npm -v
```
![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 002](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/1adef47a-e16b-437b-bb33-8dd8f7cb6eec)


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
![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 003](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9b8b6969-6342-40eb-aa1d-eccef8a23df4)



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

![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 004](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c24e3d1f-bf75-44e0-b472-f5db9316ed70)


Open port 5000 in your EC2 Security Groups.

**ACCESS THE SERVER**

Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:

```http
http://<PublicIP-or-PublicDNS>:5000
```

Quick reminder how to get your server’s Public IP and public DNS name:
1.	You can find it in your AWS web console in EC2 details
2.	Run curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.


![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 005](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9010d49a-3993-4b8f-a96b-81596487144b)

![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 006](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7cfac22c-3dc6-4bf2-8131-bf22578ce099)

---

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

The code sets up an Express router with three routes to handle CRUD operations for a todo list:

1. **GET /todos**: Retrieves a list of todos.
2. **POST /todos**: Creates a new todo.
3. **DELETE /todos/:id**: Deletes a todo by its unique identifier.

Each route has an associated callback function for handling the respective HTTP method. The router is exported for use in other parts of the application.

**MODELS**

To integrate MongoDB with our application, we'll use Mongoose, a Node.js package that simplifies working with MongoDB. Here's how to set up the models:

1. **Install Mongoose**:
   - Change directory back to `Todo`:

     ```bash
     cd ..
     ```

   - Install Mongoose:

     ```bash
     npm install mongoose
     ```

2. **Create Models Folder**:
   - Create a new folder named `models`:

     ```bash
     mkdir models
     ```

   - Change directory into the `models` folder:

     ```bash
     cd models
     ```

3. **Create Todo Model**:
   - Inside the `models` folder, create a file named `todo.js`:

     ```bash
     touch todo.js
     ```

      ![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 007](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/cec50e81-b077-4d81-b61a-371f25432fad)


   - Open the `todo.js` file with a text editor:

     ```bash
     vim todo.js
     ```

   - Copy and paste the following code into `todo.js`:

     ```javascript
     const mongoose = require('mongoose');
     const Schema = mongoose.Schema;

     // Create schema for todo
     const TodoSchema = new Schema({
         action: {
             type: String,
             required: [true, 'The todo text field is required']
         }
     });

     // Create model for todo
     const Todo = mongoose.model('todo', TodoSchema);

     module.exports = Todo;
     ```

**Update Routes**

Now, let's update the routes in the `api.js` file to use the new model:

1. Open the `api.js` file:

   ```bash
   vim api.js
   ```

2. Delete the existing code inside `api.js`:

   ```bash
   :%d
   ```

3. Paste the following code into `api.js`:

   ```javascript
   const express = require ('express');
   const router = express.Router();
   const Todo = require('../models/todo');

   router.get('/todos', (req, res, next) => {
       // Retrieve all data, exposing only the id and action field to the client
       Todo.find({}, 'action')
           .then(data => res.json(data))
           .catch(next);
   });

   router.post('/todos', (req, res, next) => {
       if (req.body.action) {
           Todo.create(req.body)
               .then(data => res.json(data))
               .catch(next);
       } else {
           res.json({
               error: "The input field is empty"
           });
       }
   });

   router.delete('/todos/:id', (req, res, next) => {
       Todo.findOneAndDelete({ "_id": req.params.id })
           .then(data => res.json(data))
           .catch(next);
   });

   module.exports = router;
   ```

---

**MONGODB DATABASE**

Now, with the models and routes set up, we're ready to integrate MongoDB into our application. 

MongoDB Database
mLab provides MongoDB database as a service solution (DBaaS). MongoDB has two cloud database management system components: mLab and Atlas, Both were formerly cloud databases managed by MongoDB. MongoDB merged the two cloud databases and as such, mLab.com redirects to the MongoDB Atlas website.


1. Create a MongoDB database and collection inside mLab

![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 008](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/14ae20ee-6e80-4bf3-9a6a-d7d8799a699b)

 ![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 009](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/fc3ffd5f-3d61-493b-8308-90b7491017d4)




 
2. Create a file in your Todo directory and name it .env, open the file
touch .env && vim .env

Add connection string below to access the database
DB = ‘mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority’

![Aspose Words a74d693a-7668-484f-a154-a3b635982d76 010](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2428e62a-826e-4bd1-926d-7dcc17e08e7a)


 
3. Update the index.js to reflect the use of .env so that Node.js can connect to the database.
vim index.js

Delete existing content in the file, and update it with the entire code below:

 ```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

// Connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log(`Database connected successfully`))
  .catch(err => console.log(err));

// Since mongoose promise is deprecated, we override it with Node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
  console.log(err);
  next();
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
 ```
 
Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.


4. Start your server using the command

```
node index.js
```

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/4736478a-a64f-405b-9947-7dfa2a0b497d)

---

**Testing Backend Code without Frontend using RESTful API**

We've completed the backend part of our To-Do application and configured a database. However, we're still missing the frontend UI. To achieve that, we need to write ReactJS code. During development, we need a way to test our code using RESTful API. Therefore, we'll use an API development client to facilitate testing.

We can use Postman

It's essential to test all the API endpoints thoroughly to ensure they are functioning correctly. For endpoints that require a body, we'll send JSON data with the necessary fields, as that's what we've set up in our code.

To get started, we'll open Postman and create a POST request to the API. This request will send a new task to our To-Do list so that the application can store it in the database.

The endpoint for this request is:

```
http://<PublicIP-or-PublicDNS>:5000/api/todos
```
### Open Postman and Set the header

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a5c93f5f-a581-4757-a830-07dbb0e576f1)

### Create POST requests to the API

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a7132e46-61f8-4d6e-8ead-dabcfc004cbe)

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5894e0a7-d259-40e9-8575-794fe6f8561a)

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5a6fd761-9a3e-4e0c-808b-84d41e61a085)

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/caa9aacf-c1a1-4b7f-951f-806a97e17a5d)
 
### Check Database Collections

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2b18b70c-e5e6-402c-8fe3-2bec7188f491)

 ### GET requests to the API

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5c935b80-d3ef-4ab3-96a4-d6a7b8d89600)

 ### DELETE requests to the API

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9059b21b-815f-46f6-ac3d-68b037dd0ab3)

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2d1cb45c-ba8b-4583-8f62-9469ca612d30)

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8bbad752-5ab5-49d2-b148-7fe0a0213d05)




Now that we've completed building the backend and API functionalities, it's time to develop a user interface for our web application to interact with the backend via API calls. To kick off the frontend development of the Todo app, we'll utilize the `create-react-app` command to set up the initial structure of our application.

## Frontend Creation

1. **Create React App**:
   - Use `create-react-app` to initialize a new React project.
   
     ```bash
     npx create-react-app client
     ```
 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b8bc5f51-2bd7-49ae-8165-0b282f15839a)


2. **Install Dependencies**:
   - Install `concurrently` to run multiple commands simultaneously.
   
     ```bash
     npm install concurrently --save-dev
     ```

   - Install `nodemon` to monitor and automatically restart the server when changes are made.
   
     ```bash
     npm install nodemon --save-dev
     ```

3. **Update `package.json`**:
   - Define three scripts in the `package.json` file:
     - `start`: to start the server.
     - `start-watch`: to start the server with nodemon for automatic restarts.
     - `dev`: to run both the server and the React app concurrently during development.
   
     ```json
     "scripts": {
         "start": "node index.js",
         "start-watch": "nodemon index.js",
         "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
     }
     ```

4. **Configure Proxy**:
   - Set the proxy in the `package.json` file of the React app to proxy any unknown requests to the backend server.
   
     ```json
     "proxy": "http://localhost:5000"
     ```

 ![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/3410beca-0970-43cf-8879-5227b0b135f0)


5. **Run the Application**:
   - Start both the server and the React app concurrently using the `npm run dev` command.
   
     ```bash
     npm run dev
     ```
![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dcef3393-57ef-4111-bd35-934e31dc32e4)

 
Remember to ensure that your EC2 instance's security group allows inbound traffic on port 3000 to access the application from the internet. This step is crucial for deploying your application and making it accessible to users.

Creating your React Components

1. **Navigate to Client Directory and Create Components Directory:**
   ```bash
   cd client
   mkdir src/components
   ```

2. **Create Input.js, ListTodo.js, and Todo.js Files:**
   ```bash
   cd src/components
   touch Input.js ListTodo.js Todo.js
   ```

3. **Open Input.js and Add Component Code:**
   ```bash
   vi Input.js
   ```
   Copy and paste the provided code into Input.js.

```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {
  state = {
    action: ""
  }

  handleChange = (event) => {
    this.setState({ action: event.target.value });
  }

  addTodo = () => {
    const task = { action: this.state.action };

    if (task.action && task.action.length > 0) {
      axios.post('/api/todos', task)
        .then(res => {
          if (res.data) {
            this.props.getTodos();
            this.setState({ action: "" });
          }
        })
        .catch(err => console.log(err));
    } else {
      console.log('Input field required');
    }
  }

  render() {
    let { action } = this.state;
    return (
      <div>
        <input type="text" onChange={this.handleChange} value={action} />
        <button onClick={this.addTodo}>add todo</button>
      </div>
    );
  }
}

export default Input;
```
4. To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.
Move to the client folder
cd ..

*Install Axios*

 ```
npm install axios
```

5. **Open ListTodo.js and Add Component Code:**
   ```bash
   cd src/components
   vi ListTodo.js
   ```
   Copy and paste the provided code into ListTodo.js.

```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {
  return (
    <ul>
      {
        todos && todos.length > 0 ? (
          todos.map(todo => {
            return (
              <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
                {todo.action}
              </li>
            );
          })
        ) : (
          <li>No todo(s) left</li>
        )
      }
    </ul>
  );
}

export default ListTodo;
```

6. **Open Todo.js and Add Component Code:**
   ```bash
   vi Todo.js
   ```
   Copy and paste the provided code into Todo.js.
```
import React, { Component } from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {
  state = {
    todos: []
  }

  componentDidMount() {
    this.getTodos();
  }

  getTodos = () => {
    axios.get('/api/todos')
      .then(res => {
        if (res.data) {
          this.setState({
            todos: res.data
          });
        }
      })
      .catch(err => console.log(err));
  }

  deleteTodo = (id) => {
    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if (res.data) {
          this.getTodos();
        }
      })
      .catch(err => console.log(err));
  }

  render() {
    let { todos } = this.state;
    return (
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos} />
        <ListTodo todos={todos} deleteTodo={this.deleteTodo} />
      </div>
    );
  }
}

export default Todo;
```

7. **Adjust App.js and App.css:**
   ```bash
   cd ../..
   vi App.js
   vi App.css
   ```
   Copy and paste the provided code into App.js and App.css respectively.
```

import React from 'react';
import Todo from './components/Todo';
import './App.css';

const App = () => {
  return (
    <div className="App">
      <Todo />
    </div>
  );
}

export default App;
```

```

.App {
  text-align: center;
  font-size: calc(10px + 2vmin);
  width: 60%;
  margin-left: auto;
  margin-right: auto;
}

input {
  height: 40px;
  width: 50%;
  border: none;
  border-bottom: 2px #101113 solid;
  background: none;
  font-size: 1.5rem;
  color: #787a80;
}

input:focus {
  outline: none;
}

button {
  width: 25%;
  height: 45px;
  border: none;
  margin-left: 10px;
  font-size: 25px;
  background: #101113;
  border-radius: 5px;
  color: #787a80;
  cursor: pointer;
}

button:focus {
  outline: none;
}

ul {
  list-style: none;
  text-align: left;
  padding: 15px;
  background: #171a1f;
  border-radius: 5px;
}

li {
  padding: 15px;
  font-size: 1.5rem;
  margin-bottom: 15px;
  background: #282c34;
  border-radius: 5px;
  overflow-wrap: break-word;
  cursor: pointer;
}

@media only screen and (min-width: 300px) {
  .App {
    width: 80%;
  }

  input {
    width: 100%;
  }

  button {
    width: 100%;
    margin-top: 15px;
    margin-left: 0;
  }
}

@media only screen and (min-width: 640px) {
  .App {
    width: 60%;
  }

  input {
    width: 50%;
  }

  button {
    width: 30%;
    margin-left: 10px;
    margin-top: 0;
  }
}
```


8. **Open index.css and Add Styling:**
   ```bash
   vi index.css
   ```
   Copy and paste the provided code into index.css.

```

body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  box-sizing: border-box;
  background-color: #282c34;
  color: #787a80;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New", monospace;
}
```


9. **Run the Application:**
   ```bash
   npm run dev
   ```
These steps guide you through creating React components, adding functionality, and styling the Todo application. After completing these steps, you can run the application using `npm run dev`.

![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c59022d2-5e0b-4917-b494-6d5b010b46aa)


 

At this point, the To-Do app is ready and fully functional with the functionality discussed earlier: 
<br>
•	Creating a task
<br>
•	deleting a task
<br>
•	viewing all the tasks.
<br>
The client can now be viewed in the browser
<br>
<br>

<img width="856" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/79f442e8-8332-438f-b707-0bd47e9f2bea">

Add some todos via the browser .

<img width="759" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/870b90b1-ff29-4129-ac92-a72a925707a0">


Check the Database 

<img width="551" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8854862a-fdfa-4b8e-aad6-280441b4188e">

<br>

# AND WE ARE FINALLY DONE 🎉🎉🎉 
 

