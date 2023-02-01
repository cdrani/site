# DevOps PBL: MEAN Stack

#### MEAN Stack is a combination of the following components:

1. [**M**ongoDB](https://www.mongodb.com/) (Document database) – Storage and retrieval of data
    
2. [**E**xpress](https://expressjs.com/) (Back-end application framework) – Makes requests to Database for Reads and Writes.
    
3. [**A**ngular](https://angular.io/) (Front-end application framework) – Handles Client and Server Requests
    
4. [**N**ode.js](https://nodejs.org/en/) (JavaScript runtime environment) – Accepts requests and displays results to end user
    

To integrate the above technologies let's build a book register:

### NodeJS Setup

```bash
sudo apt update -y
curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
sudo apt update -y # might be necessary to update source index
sudo apt install -y nodejs
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675198039983/86d28b4d-55ff-4c81-93be-37bff03b4e02.png align="center")

After installing NodeJS let's create the project directory, run `npm init` in it and create the folder structure:

```bash
sudo mkdir Book; cd Book; sudo npm init -y
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675198208177/a51073ef-9232-4a4f-9b1f-2276a9de4f0a.png align="center")

We should now have a `package.json` file inside of the `Book` directory. At the end of the project we should have a folder structure like this:

```markdown
├── Book
│   ├── apps
│   │   ├── routes.js
│   │   ├── models
│   │   │   ├── book.js
├── public
│   ├── index.js
│   ├── index.html
├── node_modules
├── server.js
├── package.json
└── package-lock.json
```

### Express Setup

Express is a minimal and flexible `Node.js` web application framework that provides features for web and mobile applications. We will use Express to pass book information to and from our MongoDB database.

```bash
sudo npm i express body-parser
```

```javascript
// ~/Book/server.js

const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```

Additionally, we need to include port `3300` in our EC2 instance's security group to be able to access the express server.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675200841698/52fcdb5e-1d33-4668-ab06-71a6b98ccfbf.png align="center")

### MongoDB Setup

MongoDB stores data in flexible, `JSON-like` documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain a book's name, ISBN, author, and number of pages.

### Installation

Let's install the latest version of the MongoDB community edition. The commands below are pulled from the official docs:

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt update
sudo apt install -y mongodb-org
sudo systemctl daemon-reload
sudo systemctl start mongod
```

We should see the service running now:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675200901322/12cb313c-a126-4e3c-8840-cf5ec68a1141.png align="center")

### Mongoose Setup

We will use Mongoose to establish a schema for the database to store data of our book register. Let's create the schema for our book model with fields for name, ISBN, author, and pages:

```javascript
// ~/Book/apps/models/book.js

const mongoose = require('mongoose');
const dbHost = 'mongodb://localhost:27017/test';

mongoose.connect(dbHost);
mongoose.connection;

mongoose.set('debug', true);

const bookSchema = mongoose.Schema({
  name: String,
  isbn: { type: String, index: true },
  author: String,
  pages: Number
});

const Book = mongoose.model('Book', bookSchema);

module.exports = mongoose.model('Book', bookSchema);
```

### Express Routes

We need to define routes for which our express server should handle when a request is made to it. The following lists `/book` route for requests for GET & POST, as well as a `/book/:isbn` to delete a book record when a DELETE request is provided with an `isbn` value for that book.

```javascript
// ~/Book/apps/routes.js

const Book = require('./models/book');

module.exports = function(app) {
    // Returns all books
    app.get('/book', function(req, res) {
        Book.find({}, function(err, result) {
            if ( err ) throw err;
            res.json(result);
        });
    }); 

    app.post('/book', function(req, res) {
        // Adds book to book collection
        const { name, isbn, author, pages } = req.body;
        const book = new Book({ name, isbn, author, pages });
        book.save(function(err, result) {
            if (err) throw err;
            res.json( {
                message:"Successfully added book",
                book: result
            });
        });
    });

    app.delete("/book/:isbn", function(req, res) {
        // Find book based on isbn value and remove from collection
        Book.findOneAndRemove(req.query, function(err, result) {
            if (err) throw err;
            res.json( {
                message: "Successfully deleted the book",
                book: result
            });
        });
    });

    const path = require('path');

    app.get('*', function(req, res) {
        res.sendfile(path.join(__dirname + '/public', 'index.html'));
    });
};
```

## AngularJS Setup

[AngularJS](https://angularjs.org) provides a web framework for creating dynamic views in your web applications. We need an `index.html` to present our data and a `index.js` to handle user input for adding or removing books from our records.

```bash
sudo npm i angularjs
```

```xml
<!-- ~/Book/public/index.html -->

<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="index.js"></script>
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
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```

Now for the interactivity connection to the UI:

```javascript
// ~/Book/public/script.js

const app = angular.module('myApp', []);

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
        $http( {
            method: 'DELETE',
            url: '/book/:isbn',
            params: { 'isbn': book.isbn }
        }).then(function successCallback(response) {
            console.log(response);
        }, function errorCallback(response) {
            console.log('Error: ' + response);
        });
    };

    $scope.add_book = function() {
        const body = {
            name: $scope.Name,
            isbn: $scope.Isbn,
            author: $scope.Author,
            pages: $scope.Pages,
        };

        $http({
            method: 'POST',
            url: '/book',
            data: body
        }).then(function successCallback(response) {
            console.log(response);
        }, function errorCallback(response) {
            console.log('Error: ' + response);
        });
    };
});
```

Our app is now complete with simple CRD (Create, Read, Delete) functionality. Start the express server with `node index.js` in the `~/Book` directory.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675201001607/bd7eff44-1ea5-45b9-a633-c2b0b1dbd3ef.png align="center")

Opening the app on our public IP address on port `3300` (for ex: `http://54.213.246.86:3300`) should display the below UI:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675201811877/f57a6108-4977-435e-8b80-35a2682736ff.png align="center")

The below image shows insertion of a book into the collection and a query to return all books:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675201767631/cdb3118b-2a34-4571-903e-eea323d27e25.png align="center")