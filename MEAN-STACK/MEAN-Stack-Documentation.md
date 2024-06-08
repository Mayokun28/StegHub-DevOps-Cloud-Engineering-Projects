
# Introduction

The MEAN stack is a popular JavaScript stack used for building web applications. It stands for MongoDB, Express.js, AngularJS (or Angular), and Node.js. Here's a brief overview of each component:

**MongoDB**: MongoDB is a NoSQL database that stores data in a flexible, JSON-like format. It is a popular choice for web applications because of its scalability, flexibility, and performance.

**Express.js**: Express.js is a web application framework for Node.js. It provides a set of features for building web applications and APIs, including routing, middleware support, and templating engines. Express.js simplifies the process of building web applications with Node.js by providing a high-level abstraction over the HTTP protocol.

**AngularJS (or Angular)**: AngularJS is a JavaScript framework maintained by Google for building dynamic web applications. It extends HTML with additional attributes and provides a set of built-in directives for creating interactive user interfaces. AngularJS follows the Model-View-Controller (MVC) architecture, making it easy to organize code and build complex web applications.

**Node.js**: Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. It allows developers to run JavaScript code outside of a web browser, making it possible to build server-side applications with JavaScript. Node.js is known for its event-driven, non-blocking I/O model, which makes it well-suited for building scalable and high-performance web applications.

## Step 0: Prerequisites

1. EC2 Instance of t2.micro type and Ubuntu 22.04 LTS (HVM) was launched in the us-east-1 region using the AWS console.

    ![alt text](mean1.JPG)

    ![alt text](mean2.JPG)

2. The security group was configured with the following inbound rules:

- Allow traffic on port 80 (HTTP) with source from anywhere on the internet.
- Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.
- Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.
- Allow traffic on port 5000 with source from any IP address.
- Allow traffic on port 3000 with source from any IP address.

    ![alt text](mean3.JPG)

3. Let's connect our instance using SSH. First, we have to cd into the folder where the private-key was downloaded (downloads). The private ssh key permission was changed for the private key file and then used to connect to the instance by running:

    ```
    cd Downloads

    chmod 400 MEAN-KEY.pem

    ssh -i "MEAN-KEY.pem" ubuntu@54.237.134.94
    ```

    ![alt text](mean4.JPG)

## Step 1 - Install Nodejs 

Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

1. Update and upgrade ubuntu
    ```
    sudo apt update
    sudo apt upgrade -y
    ```

    ![alt text](mean5.JPG)

2. Add certificates.

    ```
    sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
    ```

    ![alt text](mean6.JPG)

    ```
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    ```
    ![alt text](mean7.JPG)


3. Install NodeJS

    ```
    sudo apt-get install -y nodejs
    ```

    ![alt text](mean8.JPG)


## Step 2 - Install MongoDB

For this application, Book records were added to MongoDB that contain book name, isbn number, author, and number of pages.

1. Download the MongoDB public GPG key

    ```
    curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
    ```

2. Add the MongoDB repository.

    ```
    echo "deb [ signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
    ```

    ![alt text](mean9.JPG)

    
3. Update the package database and install MongoDB.

    ```
   sudo apt-get update
   ``` 
    ![alt text](mean10.JPG)

    ```
    sudo apt-get install -y mongodb-org
    ```

    ![alt text](mean11.JPG)

4.  Start and enable MongoDB. Also, verify that the service is up and running.

    ```
    sudo systemctl start mongod
    ```
    ```
    sudo systemctl enable mongod
    ```
    ```
    sudo systemctl status mongod
    ```

    ![alt text](mean12.JPG)

5. Install body-parser package.

    body-parser package is needed to help process JSON files passed in requests to the server.

    ```
    sudo npm install body-parser
    ```
    ![alt text](mean13.JPG)

6. Create the project root folder named ‘Books’

    ```
    mkdir Books && cd Books
    ```

    In the Books directory, initialize npm project.

    ```
    npm init
    ```
    ![alt text](mean14.JPG)

    Add file named server.js to Books folder.
    
    ```
    vim server.js
    ```

    Copy and paste the web server code below into the server.js file.

    ```
            var express = require('express');
            var bodyParser = require('body-parser');
            var app = express();
            app.use(express.static(__dirname + '/public'));
            app.use(bodyParser.json());
            require('./apps/routes')(app);
            app.set('port', 3300);
            app.listen(app.get('port'), function() {
                console.log('Server up: http://localhost:' + app.get('port'));
            });
    ```

    ![alt text](mean15-1.JPG)


## Step 3: Install Express and set up routes to the server.

Express was used to pass book information to and from our MongoDB database. Mongoose package provides a straightforward schema-based solution to model the application data. Mongoose was used to establish a schema for the database to store data of the book register.

1. Install express and mongoose

    ```
    sudo npm install express mongoose
    ```
    ![alt text](mean16.JPG)

2. In Books folder, create a folder named ‘apps’

    ```
    mkdir apps && cd apps
    ```

    In apps, create a file named routes.js

    ```
    vi routes.js
    ```

    Then paste this code inside.

    ```
        var Book = require('./models/book');
        var path = require('path');

        module.exports = function(app) {
            app.get('/book', async function(req, res) {
                try {
                    const result = await Book.find({});
                    res.json(result);
                } catch (err) {
                    res.status(500).send(err);
                }
            });

            app.post('/book', async function(req, res) {
                try {
                    var book = new Book({
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
                    res.status(500).send(err);
                }
            });

            app.delete("/book/:isbn", async function(req, res) {
                try {
                    const result = await Book.findOneAndRemove({ isbn: req.params.isbn });
                    res.json({
                        message: "Successfully deleted the book",
                        book: result
                    });
                } catch (err) {
                    res.status(500).send(err);
                }
            });

            app.get('*', function(req, res) {
                res.sendFile(path.join(__dirname + '/public', 'index.html'));
            });
        };
    ```

    ![alt text](mean17-1.JPG)

3. In the ‘apps’ folder, create a directory named models.

    ```
    mkdir models && cd models
    ```
    In models, create a file named book.js

    ```
    vim book.js
    ```

    ![alt text](mean18.JPG)


    Copy and paste the code below into book.js

    ```
        var mongoose = require('mongoose');
        var dbHost = 'mongodb://localhost:27017/test';
        mongoose.connect(dbHost, { useNewUrlParser: true, useUnifiedTopology: true });
        mongoose.connection;
        mongoose.set('debug', true);

        var bookSchema = mongoose.Schema({
            name: String,
            isbn: { type: String, index: true },
            author: String,
            pages: Number
        });

        var Book = mongoose.model('Book', bookSchema);

        module.exports = Book;

    ```

    ![alt text](mean19-1.JPG)

## Step 4 - Access the routes with AngularJS.

Angular JS provides a web framework for creating dynamic views in your web applications.
In this project, AngularJS was used to connect the web page with Express and perform actions on the book register.

1. Change the directory back to ‘Books’ and create a folder named ‘public’

    ```
    cd ../.. 
    ```
    ```
    mkdir public && cd public
    ```

    Create a file named script.js inside the 'public' folder.
    
    ```
    vim script.js
    ```

    and copy this code inside.

    ```
    var app = angular.module('myApp', []);
    app.controller('myCtrl', function($scope, $http) {
        $http({
            method: 'GET',
            url: '/book'
        }).then(function successCallback(response) {
            $scope.books = response.data;
        }, function errorCallback(response) {
        console.log('Error: ' + response);
        });

        $scope.del_book = function(book) {
            $http({
                method: 'DELETE',
                url: '/book/' + book.isbn,
            }).then(function successCallback(response) {
                console.log(response);
            }, function errorCallback(response) {
            console.log('Error: ' + response);
            });
        };

        $scope.add_book = function() {
            var body = {
                name: $scope.Name,
                isbn: $scope.Isbn,
                author: $scope.Author,
                pages: $scope.Pages
            };
            $http({
                method: 'POST',
                url: '/book',
                data: body,
                headers: { 'Content-Type': 'application/json' }
            }).then(function successCallback(response) {
            console.log(response);
            }, function errorCallback(response) {
                console.log('Error: ' + response);
            });
        };
    });  
    ```

    ![alt text](mean20-1.JPG)

2.  In ‘public’ folder, create a file named index.html

    ```
    vim index.html
    ```

    Copy and paste the code below into the index.html file.

```
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
  <script src="script.js"></script>
  <style>
    /* Add your custom CSS styles here */
  </style>
</head>
<body>
  <div>
    <table>
      <tr>
        <td>Name:</td>
        <td><input type="text" ng-model="Name"></td>
      </tr>
      <tr>
        <td>Isbn:</td>
        <td><input type="text" ng-model="Isbn"></td>
      </tr>
      <tr>
        <td>Author:</td>
        <td><input type="text" ng-model="Author"></td>
      </tr>
      <tr>
        <td>Pages:</td>
        <td><input type="number" ng-model="Pages"></td>
      </tr>
    </table>
    <button ng-click="add_book()">Add</button>
    <div ng-if="successMessage">{{ successMessage }}</div>
    <div ng-if="errorMessage">{{ errorMessage }}</div>
  </div>
  <hr>
  <div>
    <table>
      <tr>
        <th>Name</th>
        <th>Isbn</th>
        <th>Author</th>
        <th>Page</th>
        <th>Action</th>
      </tr>
      <tr ng-repeat="book in books">
        <td>{{ book.name }}</td>
        <td>{{ book.isbn }}</td>
        <td>{{ book.author }}</td>
        <td>{{ book.pages }}</td>
        <td><button ng-click="del_book(book)">Delete</button></td>
      </tr>
    </table>
  </div
</body>
</html>
```

![alt text](mean21.JPG)

3. Change the directory back up to ‘Books’ and start the server.

    ```
    cd ..
    ```

    ```
    node server.js
    ```

    ![alt text](mean22.JPG)

    The server is now up and running, Connection to it is via port 3300. Note: ensure you adjust your inbound rule on your server and open port 3300.

4. Access the Book Register web application from the internet with a browser using the Public IP address or Public DNS name.

    ```
    http://54.237.134.94:3300
    ```
    ![alt text](mean23.JPG)

    The output above shows our book register web application is up and running.

    You can add more books to the register.

    ![alt text](image-10.png)

## Conclusion

Congratulations! We have successfully set up and deployed a simple Book Register web form using the MEAN stack on AWS EC2 instance. This web form allows you to add, view, and delete books from a MongoDB database through a web interface powered by AngularJS.

The MEAN stack comprising MongoDB, Express.js, AngularJS (or Angular), and Node.js provides a powerful and cohesive set of technologies for building modern web applications.




