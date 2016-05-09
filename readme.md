# A Simple TODO app using Express, Mongoose, and MongoDB

This app was developed as part of the General Assembly WDI class in Atlanta, GA.

* Author: Dr. Mike Hopper
* Date: February, 2016

# Steps To Reproduce

* [Step 1 - Create the Project](#step-1---create-the-project)
* [Step 2 - Create a model and seeds file](#step-2---create-a-model-and-seeds-file)
* [Step 3 - Create the routes and controller logic for our 7 RESTfull endpoints](#step-3---create-the-routes-and-controller-logic-for-our-7-restfull-endpoints)
* [Step 4 - Create the TODO Views](#step-4---create-the-todo-views)
* [Lab Time](#lab-time)

## Step 1 - Create the Project

1a. Use the `express generator` to generate the project

```bash
express --ejs todos
cd todos && npm install
```

1b. Add `mongoose` to the project

```bash
npm install --save mongoose
```

Edit `app.js` and add the following lines:

```javascript
var mongoose = require('mongoose');
...
// Connect to database
mongoose.connect('mongodb://localhost/todos');
```

1c. Edit `package.json` and add nodemon

```json
  "scripts": {
    "start": "nodemon ./bin/www"
  },
```


1e. Create git repository for project:

```bash
echo "node_modules" > .gitignore
git init
git add -A
git commit -m "Initial commit."
git tag step1
```

## Step 2 - Create a model and seeds file

2a. Create the `Todo` mongoose model:

```bash
mkdir models
touch models/todo.js
```

```javascript
var mongoose = require('mongoose');

var TodoSchema = new mongoose.Schema({
  title: { type: String, required: true },
  completed: { type: Boolean, required: true }
});

module.exports = mongoose.model('Todo', TodoSchema);
```

2b. Create a `seeds.js` script

```bash
touch seeds.js
```

Add the following to `seeds.js`:

```javascript
var mongoose = require('mongoose');
var Todo = require('./models/todo');

mongoose.connect('mongodb://localhost/todos');

// our script will not exit until we have disconnected from the db.
function quit() {
  mongoose.disconnect();
  console.log('\nQuitting!');
}

// a simple error handler
function handleError(err) {
  console.log('ERROR:', err);
  quit();
  return err;
}

console.log('removing old todos...');
Todo.remove({})
.then(function() {
  console.log('old todos removed');
  console.log('creating some new todos...');
  var groceries  = new Todo({ title: 'groceries',    completed: false });
  var feedTheCat = new Todo({ title: 'feed the cat', completed: true });
  // return groceries.save();
  return Todo.create([groceries, feedTheCat]);
})
.then(function(savedTodos) {
  console.log('Todos have been saved');
  return Todo.find({});
})
.then(function(allTodos) {
  console.log('Printing all todos:');
  allTodos.forEach(function(todo) {
    console.log(todo);
  });
  quit();
});
```

2c. Run the `seeds` script:

```bash
node seeds.js
```

2d. Save your work:

```bash
git add -A
git commit -m "Added Todo model and seeds script."
git tag step2
```

## Step 3 - Create the routes and controller logic for our 7 RESTfull endpoints

3a. Add methodOverride to our project:

```bash
npm install --save method-override
```

Edit `app.js`:

```javascript
var methodOverride = require('method-override');

...

app.use(methodOverride('_method'));
```

3b. Create `routes/todos.js` with the following content:

```bash
touch routes/todos.js
```

Add the following to `routes/todos.js`:

```javascript
var express = require('express');
var router = express.Router();
var Todo = require('../models/todo');

function makeError(res, message, status) {
  res.statusCode = status;
  var error = new Error(message);
  error.status = status;
  return error;
}

// INDEX
router.get('/', function(req, res, next) {
  // get all the todos and render the index view
  Todo.find({})
  .then(function(todos) {
    res.render('todos/index', { todos: todos } );
  }, function(err) {
    return next(err);
  });
});

// NEW
router.get('/new', function(req, res, next) {
  var todo = {
    title: '',
    completed: false
  };
  res.render('todos/new', { todo: todo } );
});

// SHOW
router.get('/:id', function(req, res, next) {
  Todo.findById(req.params.id)
  .then(function(todo) {
    if (!todo) return next(makeError(res, 'Document not found', 404));
    res.render('todos/show', { todo: todo });
  }, function(err) {
    return next(err);
  });
});

// CREATE
router.post('/', function(req, res, next) {
  var todo = new Todo({
    title: req.body.title,
    completed: req.body.completed ? true : false
  });
  todo.save()
  .then(function(saved) {
    res.redirect('/todos');
  }, function(err) {
    return next(err);
  });
});

// EDIT
router.get('/:id/edit', function(req, res, next) {
  Todo.findById(req.params.id)
  .then(function(todo) {
    if (!todo) return next(makeError(res, 'Document not found', 404));
    res.render('todos/edit', { todo: todo });
  }, function(err) {
    return next(err);
  });
});

// UPDATE
router.put('/:id', function(req, res, next) {
  Todo.findById(req.params.id)
  .then(function(todo) {
    if (!todo) return next(makeError(res, 'Document not found', 404));
    todo.title = req.body.title;
    todo.completed = req.body.completed ? true : false;
    return todo.save();
  })
  .then(function(saved) {
    res.redirect('/todos');
  }, function(err) {
    return next(err);
  });
});

// DESTROY
router.delete('/:id', function(req, res, next) {
  Todo.findByIdAndRemove(req.params.id)
  .then(function() {
    res.redirect('/todos');
  }, function(err) {
    return next(err);
  });
});

module.exports = router;
```

3c. Edit `app.js` and add the following code:

```javascript
var todosRouter = require('./routes/todos');
...
app.use('/todos', todosRouter);
```

3d. Save your work:

```bash
git add -A
git commit -m "Created Todo routes and controller logic."
git tag step3
```

## Step 4 - Create the TODO Views

```bash
mkdir views/partials
touch views/partials/head.ejs
touch views/partials/header.ejs
touch views/partials/footer.ejs

mkdir views/todos
touch views/todos/_form.ejs
touch views/todos/index.ejs
touch views/todos/show.ejs
touch views/todos/new.ejs
touch views/todos/edit.ejs
```

4a. Add the following content to `views/partials/head`:

```html
<meta charset="UTF-8">
<title>TODOs</title>

<link rel="stylesheet" type="text/css" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
<link rel="stylesheet" type="text/css" href="/stylesheets/style.css">
```

4b. Add the following content to `views/partials/header`:

```html
<nav class="navbar navbar-default" role="navigation">
  <div class="container-fluid">
    <div class="navbar-header">
      <a class="navbar-brand" href="#">
        <span class="glyphicon glyphicon glyphicon-tree-deciduous"></span>TODOs
      </a>
    </div>

    <ul class="nav navbar-nav">
      <li><a href="/">Home</a></li>
      <li><a href="/todos">TODOs</a></li>
    </ul>
  </div>
</nav>
```

4c. Add the following content to `views/partials/footer`:

```html
<p class="text-center text-muted">&copy; Copyright 2016 ATLANTA WDI</p>
```

4d. Add the following content to `views/todos/index.ejs`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <% include ../partials/head %>
</head>

<body class="container-fluid">
  <header>
    <% include ../partials/header %>
  </header>

  <main>
    <div>
      <h3>TODOs:</h3>
      <% todos.forEach(function(todo) { %>
        <h4>
          <form method="POST" action="/todos/<%= todo._id %>?_method=DELETE">
            <a href="/todos/<%= todo._id %>">
              <%= todo.title + ' - ' + todo.completed %>
            </a>
            <button type="submit" class="btn btn-xs btn-danger">Delete</button>
          </form>
        </h4>
      <% }) %>
    </div>
    <a href="/todos/new" class="btn btn-primary">New</a>
  </main>

  <footer>
    <% include ../partials/footer %>
  </footer>
</body>
</html>
```

4e. Add the following content to `views/todos/show.ejs`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <% include ../partials/head %>
</head>

<body class="container-fluid">
  <header>
    <% include ../partials/header %>
  </header>

  <main>
    <div>
      <h3>SHOW</h3>
      <p><b>Title: </b><%= todo.title %></p>
      <p><b>ID: </b><%= todo._id %></p>
      <p><b>Completed: </b><%= todo.completed %></p>
    </div>

    <a href="/todos" class="btn btn-primary">Back</a>
    <a href="/todos/<%= todo._id %>/edit" class="btn btn-danger">Edit</a>
  </main>

  <footer>
    <% include ../partials/footer %>
  </footer>
</body>
</html>
```

4f. Add the following content to `views/todos/_form.ejs`:

```html
<div class="form-group">
  <label for="title">Title</label>
  <input type="text"
         class="form-control"
         name="title"
         value="<%= todo.title %>">
</div>

<div class="form-group">
  <label for="completed">Completed</label>
  <input type="checkbox"
         class="form-control"
         name="completed"
         <%= todo.completed ? 'checked' : '' %> >
</div>
```

4g. Add the following content to `views/todos/new.ejs`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <% include ../partials/head %>
</head>

<body class="container-fluid">
  <header>
    <% include ../partials/header %>
  </header>

  <main>
    <h3>NEW</h3>
    <form class="todo-form" action="/todos" method="post">
      <% include _form %>
      <a href="/todos" class="btn btn-primary">Back</a>
      <button type="submit" class="btn btn-success">Save</button>
    </form>
  </main>

  <footer>
    <% include ../partials/footer %>
  </footer>
</body>
</html>
```

4h. Add the following content to `views/todos/edit.ejs`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <% include ../partials/head %>
</head>

<body class="container-fluid">
  <header>
    <% include ../partials/header %>
  </header>

  <main>
    <h3>EDIT</h3>

    <form class="todo-form" action="/todos/<%= todo._id %>?_method=PUT" method="post">
      <% include _form %>
      <a href="/todos/<%= todo._id %>" class="btn btn-primary">Back</a>
      <button type="submit" class="btn btn-success">Save</button>
    </form>
  </main>

  <footer>
    <% include ../partials/footer %>
  </footer>
</body>
</html>
```

4i. Edit `views/index.ejs` and set it's content to:

```html
<!doctype html>
<html lang="en">
<head>
  <% include partials/head %>
</head>

<body class="container-fluid">
  <header>
    <% include partials/header %>
  </header>

  <main>
    <div class="jumbotron">
      <h1>Welcome to our TODOs App</h1>
  </main>

  <footer>
    <% include partials/footer %>
  </footer>

 </body>
</html>
```

4j. Test it out:

```bash
echo '#!/bin/bash' > run.bash
echo 'DEBUG=todos:* npm start' >> run.bash
chmod u+x run.bash
./run.bash
```

4k. Save your work:

```bash
git add -A
git commit -m "Created Todo views."
git tag step4
```

## Lab Time

Add the following features to the TODOs app:

* Add the properties `createdAt` and `lastUpdatedAt` to the `Todo` schema with a type of `Date`.  Set the `createdAt` value in the `CREATE` logic of the TODO router and set the `lastUpdatedAt` value in the `UPDATE` logic of the TODO router. Add the display of these values to the TODO's `show.ejs` view.

* Convert the display of the `completed` field to a checkbox using either the HTML checkbox input control or using icons such as the Twitter Bootstrap's `glyphicon-ok` and `glyphicon-unchecked` icons.

* Add a toggle feature to the Todo Index view so that you can complete or un-complete a Todo from the index page. For this feature you will need to add a new route to the Todo router, such as `GET /todos/:id/toggle`.
