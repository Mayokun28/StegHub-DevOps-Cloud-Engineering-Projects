# Deploying a Web Solution using MERN STACK ON EC2

The MERN stack is a popular technology stack used for building dynamic and scalable web applications. MERN is an acronym that stands for MongoDB, Express.js, React.js, and Node.js. Each of these components provides a layer in the stack, handling everything from the database, server, and API layers to the frontend.  It leverages a combination of JavaScript technologies:

MongoDB: A NoSQL document database for flexible data storage.

Express.js: A lightweight web framework for Node.js that simplifies building APIs and web applications.

React.js: A powerful library for creating interactive user interfaces.

Node.js: A JavaScript runtime environment that allows execution of server-side JavaScript code.

This guide provides a comprehensive overview of setting up and utilizing each component of the MERN stack, to develop robust web applications instance.

Workflow of MERN Stack
1. Client Requests: The process begins with the client sending a request to the server, typically through a web interface created with React.js.
2. Server Interaction: Express.js running on Node.js handles the incoming request. It can interact with the database to create, read, update, or delete data.
3. Database Operations: MongoDB stores or retrieves data, which is then sent back to the server.
Response Generation: The server may perform additional processing based on the data retrieved or manipulated in the database before sending a response back to the client.
4. Displaying Data: React.js then takes this data and updates the view for the user, without needing a full page refresh.

# Step 0: Prerequisites

1. EC2 Instance of t3.small type and Ubuntu 24.04 LTS (HVM) was lunched in the us-east-1 region using the AWS console. The choice of the t3.small instance type was because:

- Memory: The t3.small instance offers more memory than the t2.micro, which is advantageous for applications that require more memory to operate efficiently.

- Burst Capability: While both instances offer burstable CPU performance, the t3 instances have a more flexible burst model, allowing for more sustained performance during burst periods. This is important for workloads that require consistent performance over longer periods.

- Performance: While both instances offer burstable performance, the t3.small typically provides better baseline performance compared to the t2.micro. This might be necessary for applications that require a bit more processing power.

    ![performance](/MERN-STACK/Images/1.JPG)

    ![Performance2](/MERN-STACK/Images/2.JPG)

2. The security group was configured with the following inbound rules:

- Allow traffic on port 80 (HTTP) with source from anywhere on the internet.

- Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.

- Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.

- Allow traffic on port 5000 (Custom TCP) with source from any IP address.

    ![Traffic](/MERN-STACK/Images/3.JPG)


3. Let's connect to our instance using SS. First, we will `cd` into the folder where the private-key was downloaded. Then ssh to the instance by running:

    ```
    cd Downloads

    chmod 400 mern-key.pem

    ssh -i "mern-key.pem" ubuntu@50.17.88.9
    ```

    ![alt text](/MERN-STACK/Images/4.JPG)

# Step 1 - Backend Configuration

1. Update and upgrade list of packages in package manager
    ```
    sudo apt update
    sudo apt upgrade -y
    ```
    ![ackend](/MERN-STACK/Images/5.JPG)

2. Get the location of Node.js software from ubuntu repositories.

    ```
    curl fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    ```
    ![Nodejs](/MERN-STACK/Images/6.JPG)

3. Install node.js on the server with the command below.

    ```
    sudo apt-get install -y nodejs
    ```
    ![Server](/MERN-STACK/Images/7.JPG)

    Note: the above command installs both node.js and npm. NPM is a package manager for Node just as apt is a package manager for Ubuntu. It is used to install Node modules and packages and to manage dependency conflicts.

4. Verify the Node installation with the command below.
    ```
    node -v        // Gives the node version

    npm -v        // Gives the node package manager version
    ```

    ![Node](/MERN-STACK/Images/8.JPG)

## Application Code Setup   

1. Create a new directory for the To-Do task and change directory to the new directory Todo created.
    ```
        mkdir Todo
        cd Todo
    ```
    ![alt text](/MERN-STACK/Images/9.JPG)

2. Then initialize the project directory.

    ![alt text](/MERN-STACK/Images/10.JPG)

    This is to initialize the project directory and in the process, creates a new file called package.json.
    This file will contain information about your application and the dependencies it needs to run. Follow the prompts after running the command. You can press “Enter” several times to accept default values, then accept to write out the package.json file by typing yes.

## Install ExpressJs

Express is a framework for Node.js. It simplifies development and abstracts a lot of low level details. For example, express helps to define routes of your application based on HTTP methods and URLs.

1.  Install Express using npm
    ```
    npm install express
    ```
    ![alt text](/MERN-STACK/Images/11.JPG)

2. Create a file index.js and run ls to confirm the file is successfully created.

    ```
    touch index.js
    ls
    ```
    ![alt text](/MERN-STACK/Images/12.JPG)

3. Install dotenv module

    ```
    npm install express
    ```
    ![alt text](/MERN-STACK/Images/13.JPG)

4. Open index.js file

    ```
    vim index.js
    ```
    ```
    const express = require('express');

    const app = express();

    const port = process.env.PORT || 5000;

    app.use((req, res, next) => {
      res.header('Access-Control-Allow-Origin', '*');
      res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
    next();
    });

    app.use((req, res, next) => {
      res.send('Welcome to Express');
    });

    app.listen(port, () => {
      console.log(`Server running on port ${port} `);
    });
    ```
    Note: Port 5000 have been specified to be used in the code. This was required later on the browser.

    ![alt text](/MERN-STACK/Images/14.JPG)

5. Start the server to see if it works. Open your terminal in the same directory as your index.js file. Run

    ```
    node index.js
    ```
    ![alt text](/MERN-STACK/Images/15.JPG)

    Note: Ensure you have opened the port 5000 in your ec2 security group.

6. Open up your browser and try to access your server's public IP followed by port 5000.

    ```
   http://50.17.88.9:5000
   ```
   ![alt text](/MERN-STACK/Images/16.JPG)

# Step 2 - Creating the Routes

There are three actions that the ToDo application needs to be able to do:

- Create a new task
- Display list of all task
- Delete a completed task

Each task was associated with some particular endpoint and used different standard HTTP request methods: POST, GET, DELETE.

For each task, routes were created which defined various endpoints that the ToDo app depends on.

1. Create a folder routes and change directory to routes folder

    ```
    $ mkdir routes && cd routes
    ```
    ![alt text](/MERN-STACK/Images/17.JPG)


2. Create a file api.js and open the file then write the code below.

    ```
    touch api.js

    vim api.js
    ```

    ```
        const express = require('express');
        const router = express.Router();
        const Todo = require('../models/todo');

        // Get all todos
            router.get('/todos', (req, res, next) => {
        // This will return all the data, exposing only the id and action field to the client
        Todo.find({}, 'action')
            .then(data => res.json(data))
            .catch(next);
        });

        // Create a new todo
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

        // Delete a todo by id
            router.delete('/todos/:id', (req, res, next) => {
             Todo.findOneAndDelete({ "_id": req.params.id })
             .then(data => res.json(data))
            .catch(next);
        });

        module.exports = router;

    ```
    ![alt text](/MERN-STACK/Images/18.JPG)

    ![alt text](/MERN-STACK/Images/19.JPG)

    
# Models
A model is at the heart of JavaScript based applications and it is what makes it interactive.

Models was used to define the database schema. This is important in order be able to define the fields stored in each Mongodb document.

In essence, the schema is a blueprint of how the database is constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties. To create a schema and a model, mongoose was installed, which is a Node.js package that makes working with mongodb easier.

1. Change the directory back to Todo folder and install *mongoose*.

    ```
    cd ..
    npm install mongoose
    ```
   ![alt text](/MERN-STACK/Images/20.JPG)

2. Create a new folder models, schange directory to models folder, create a file todo.js inside models folder. Open the file todo.js and edit with vim editor.

    ```
    mkdir models && cd models && touch todo.js
    ```
    ![alt text](/MERN-STACK/Images/21.JPG)

    Past the code below into the file todo.js

    ```
    const mongoose = require('mongoose');
    const Schema = mongoose.Schema;

    // Create schema for todo
    const TodoSchema = new Schema({
      action: {
        type: String,
        required: [true, 'The todo text field is required'],
      }
    });

    // Create model for todo
    const Todo = mongoose.model('todo', TodoSchema);

    module.exports = Todo;
   ```

   ![alt text](/MERN-STACK/Images/22.JPG)

3. We need to update our routes in api.js to make use of the new model. In Routes directory, open api.js and delete the code inside with :%d. Paste the new code below into it:

    ```
    vim api.js
    ```
    ```
        const express = require('express');
        const router = express.Router();
        const Todo = require('../models/todo');

        router.get('/todos', (req, res, next) => {
          // This will return all the data, exposing only the id and action field to the client
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
      Todo.findOneAndDelete({"_id": req.params.id})
      .then(data => res.json(data))
      .catch(next);
    });

    module.exports = router;
    ```

# MongoDB Database

mLab provides MongoDB database as a service solution (DBaaS). MongoDB has two cloud database management system components: mLab and Atlas, Both were formerly cloud databases managed by MongoDB (MongoDB acquired mLab in 2018, with certain differences). In November, MongoDB merged the two cloud databases and as such, mLab.com redirects to the MongoDB Atlas website.

1. Sign Up to mongoDB.

2. Create a cluster, select AWS as the cloud provider and choose a region near you.

    ![alt text](/MERN-STACK/Images/24.JPG)

    AWS cloud provider, in region N. Virginia (us-east-1) was selected.

    Then your cluster is created.

    ![alt text](/MERN-STACK/Images/25-1.JPG)

3. Access from anywhere to the MongoDB database was allowed (Not secure but it is ideal for testing).

    ![alt text](/MERN-STACK/Images/25.JPG)

4. Create a database user and give it admin access. - Click on database access under the security section (left sidebar).
- Click on “Add New Database User”
- Enter a username and password. 
- Ensure the “Read and Write to any database” option is selected.
- Click "Create Database User"

    ![alt text](/MERN-STACK/Images/27.JPG)

5. A database named "todo_do" and collections named "todos" was created.

    ![alt text](/MERN-STACK/Images/28.JPG)



6. Create a file in your Todo directory and name it .env, open the file with the vim editor

    ```
    touch .env && vim .env
    ```

    Paste your database connection string below into the .env file to access the database

    ```
    DB=mongodb+srv://mayokun:mayorcloud999@mern-stack-db.uun9zqi.mongodb.net/?retryWrites=true&w=majority&appName=MERN-STACK-DB
    ```
    Make sure you use your own MongoDB URL from mLab after you created your database and user. Replace <USER> with the username and <PASSWORD> with the password of the user you created.

    ![alt text](/MERN-STACK/Images/29.JPG)


7. Update index.js to reflect the use of .env so that Node.js can connect to the DB.


    ```
    vim index.js
    ```

    Delete all the existing content in the file, and update it with the entire code below:

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

    ![alt text](/MERN-STACK/Images/30-1.JPG)

    Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.   

8. Start your Nodejs Server
    ```
    node index.js
    ```

    ![alt text](/MERN-STACK/Images/32.JPG)

# Testing Backend Code without Frontend using RESTful API

Postman was used to test the backend code. The endpoints were tested.

1. Download postman software from https://www.postman.com/downloads/

2. Open Postman and Set the header

    ```
      http://18.207.198.13:5000/api/todos
      ```  

    ![alt text](/MERN-STACK/Images/34.JPG)

    Create a POST request to the API

    This request creates a new record/task in our To-Do application.

    ![alt text](/MERN-STACK/Images/35.JPG)

    Make a GET request to the API

    This request retrieves all existing records/tasks from our To-Do application.

    ![alt text](/MERN-STACK/Images/36.JPG)

    Check Database Collections

    ![alt text](/MERN-STACK/Images/37.JPG)

# Step 2 - Frontend Creation

It is time to create a user interface for a Web client (browser) to interact with the application via API.

1. In the same root directory as your backend code, which is the Todo directory, run:

    ```
    npx create-react-app client
    ```

    ![alt text](/MERN-STACK/Images/38.JPG)

This created a new folder in the Todo directory called client, where all the react code was added.

## Running a React App

Before testing the react app, the following dependencies needs to be installed in the project root directory.

- Install concurrently. It is used to run more than one command simultaneously from the same terminal window.

    ```
    npm install concurrently --save-dev
    ```

    ![alt text](/MERN-STACK/Images/39.JPG)

- Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.

    ```
    npm install nodemon --save-dev
    ```
    ![alt text](/MERN-STACK/Images/40.JPG)

- In the Todo folder open the package.json file using the vim editor, change the highlighted part of the below screenshot and replace with the code below:

    ```
    "scripts": {
    "start": "node index.js",
    "start-watch": "nodemon index.js",
    "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
    }
    ```
   ![alt text](/MERN-STACK/Images/41.JPG)

Configure Proxy In package.json

1. Change directory to “client”

    ```
        cd client
    ```
2. Open the package.json file

    ```
    vim package.json
    ```
    ![alt text](/MERN-STACK/Images//MERN-STACK/Images/42.JPG)

3. Add the key value pair in the package.json file

    ```
    “proxy”: “http://localhost:5000”
    ```

    ![alt text](/MERN-STACK/Images/43.JPG)

The whole purpose of adding the proxy configuration above is to make it possible to access the application directly from the browser by simply calling the server url like http://locathost:5000 rather than always including the entire path like http://localhost:5000/api/todos.

Ensure you are inside the Todo directory, and simply do:

 ```
    npm run dev
```

![alt text](/MERN-STACK/Images/44.JPG)

The app opened and started running on localhost:3000

Note: In order to access the application from the internet, TCP port 3000 had been opened on EC2 security group.

![alt text](/MERN-STACK/Images/45.JPG)


## Creating React Components

One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. 

1. For the Todo app, there are two stateful components and one stateless component. From Todo directory, run:

    ```
    cd client
    ```
    Move to the “src” directory

    ```
    cd src
    ```
2. Inside your src folder, create another folder called “components”

    ```
    mkdir components
    ```

    Move into the components directory

    ```
    cd components
    ```

    ![alt text](/MERN-STACK/Images/46.JPG)

    Open Input.js file

    ```
    vim Input.js
    ```

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

    ![alt text](/MERN-STACK/Images/47.JPG)

    To make use of axios, which is a Promise-based HTTP client for the browser and Node.js, you will need to navigate to your client directory from your terminal:

    Move to the client folder

    ```
    cd ../..
    ```
    Install Axios

    ```
    npm install axios
    ```

    ![alt text](/MERN-STACK/Images/48.JPG)

    Go to components directory

    ```
    cd src/components
    ```

    After that open the ListTodo.js

    ```
    vim ListTodo.js
    ```

    Copy and paste the following code:

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

    ![alt text](/MERN-STACK/Images/49.JPG)

    Then open the Todo.js file with the vim text editor, 

    ```
    vim Todo.js
    ```

    Paste the following code:

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

    ![alt text](/MERN-STACK/Images/50.JPG)

    Adjust the React code by Deleting the logo and adjust App.js to look like this:

    Move to src folder

    ```
    cd ..
    ```
    Ensure to be in the src folder and open the App.js with the vim text editor:

    ```
    vim App.js
    ```

    ![alt text](/MERN-STACK/Images/51.JPG)

    In the src directory, open the App.css with the vim text editor:

    ```
    vim App.css
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
    ursor: pointer;
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
    ![alt text](/MERN-STACK/Images/52.JPG)

    In the src directory, open the index.css using the vim text editor.

    ```
    vim index.css
    ```

   ![alt text](/MERN-STACK/Images/53.JPG)

    copy and paste the code below into index.css file;

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

    ![alt text](/MERN-STACK/Images/54.JPG)

    Go to the Todo directory.

    ```
    $ cd  ../..
    ```

    In the Todo directory run;

    ```
    npm run dev
    ```
    ![alt text](/MERN-STACK/Images/55.JPG)

    At this point, the To-Do app is ready and fully functional with the functionality discussed earlier: Creating a task, deleting a task, and viewing all the tasks.

    The client can now be viewed in the browser using;

    http://publicIP:3000

    ![alt text](/MERN-STACK/Images/56.JPG)

    You can go ahead to add some todos via the user interface.

    ![alt text](/MERN-STACK/Images/57.JPG)

## Conclusion
We just created a todo app using the MERN stack on the cloud server. We deployed a frontend application using React.js that communicates with a backend application written using Express.js. We aldo created a MongoDB backend for storing tasks in a database. Also we were able to perform API testing using Postman.

This documentation provides a detailed guide for building and deploying a web application using a MERN stack.



