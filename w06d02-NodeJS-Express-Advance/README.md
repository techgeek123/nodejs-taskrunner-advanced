# Learning Competencies

- Advance NodeJS(Sandbox)
- Routing, Middleware etc
- Express Generators


# Introduction


## Express

Express extendes NodeJS. The request & response are actually inherited from nodeJS https module. Let's have a look at Express Code

Please read express source code for request [here](https://github.com/expressjs/express/blob/master/lib/request.js#L31)

Please read express source code for response [here](https://github.com/expressjs/express/blob/master/lib/response.js#L42)

So, a lot of things(methods, classes, modules) are that comes in Express are inherited from NodeJS. It gives us a freedom to call any NodeJS method or functioality from within our Express code

For ex.

```javascript
var express = require('express')
var app = express()

app.get('/', function(request, response){
	response.write('Hello World')            // Node Function
	response.end()                           // Node Function
})
```

Output:
```
$ curl http://localhost:3000/
Hello World
```

We can reach out to Node from Express & this is very helpful functionality when we will write extensions in Express


To serialize our datastructures(Objects) to Json we can use `response.send()` method. 

`response.json()` function behaves same as `response.send()` function for objects & arrays

```javascript
var express = require('express')
var app = express()

app.get('/blocks', function(request, response){
	blocks = ['Fixed', 'Movable', 'Static']
	response.json(blocks)
})
```

We usually don't respond with HTML from Routes in Express. Rather we use some templating library called `EJS`, `Jade`


#### Redirecting Routes to new location

```javascript
app.get('/blocks', function(request, response){
	response.redirect('/parts');
})
```

Now `/blocks` route will be redircted to `/parts`. We can also change the status code for re-direction. The default status code you get back while doing `curl -i http://localhost:3000/` is 302. However you can change the status code

```javascript
app.get('/blocks', function(request, response){
	response.redirect(301, '/parts');
})
```


#### Building your own custom Middleware

In this we will be building a `logger` middleware which displays duration of each request made to our application 

logger.js
```javascript
module.exports = function(request, response, next) {  // NodeJS Module System follows common JS Specification
	var start = +new Date() // + sign converts to number of milliseconds it lapsed since Jan 1, 1970. This is widely accepted standard in UNIX community where NODEJS & EXPRESS adapt from.

	var stream = process.stdout
	var url = request.url
	var method = request.method

	// But middlewares executes sequentially. How do they track the duration?
	// By listening to Event Emitters
	// It turns out that the response objects is an Event Emitter
	// It emits finish when response has finished completing

	response.on('finish', function()) {
		var duration = +new Date - start
		var message = method + ' to ' + url + ' \n took '  + duration + ' ms \n\n'
		stream.write(message)
	}

	next()     // moves request to next middleware in stack
}
```


app.js
```javascript
var express = require('express')
var app = express()

var logger = require('./logger')
app.use(logger)                  // add to middleware stack for use

app.use(express.static('public'))

app.get('/blocks', function(request, response){
	var blocks = ['Fixed', 'Movable', 'Rotating']
	response.json(blocks)
})

app.listen(3000, function() {
	console.log('Listening on 3000 \n')
})
```

#### Intercepting Requests before reaching express routes

We can intercept requests so that we can run some pre-conditional checks before they reach the express routes. Do some data massaging before a route kicks in. 

###### Query String Parameter

```javascript
var express = require('express')
var app = express()

app.get('/blocks', function(request, response){
	blocks = ['Fixed', 'Movable', 'Static']
	response.json(blocks)
})
```
Output: returns entire blocks datastructure


```javascript
var express = require('express')
var app = express()

app.get('/blocks', function(request, response){
	blocks = ['Fixed', 'Movable', 'Static']
	response.json(blocks)
})
```

Client Get to /blocks -------------------------> Server

['Fixed', 'Movable', 'Static'] <---------------- Server

Client Get to /blocks?limits=1 ----------------> Server

['Fixed'] <------------------------------------- Server

The code for this will look like something below

```javascript
var express = require('express')
var app = express()

app.get('/blocks', function(request, response){
	blocks = ['Fixed', 'Movable', 'Static']

	if(request.query.limit >= 0) {
		response.json(blocks.slice(0, request.query.limit))
	} else {
		response.json(blocks)
	}
})
```


`$ curl -i http://localhost:3000/limit=1`

Output: ['Fixed']

`$ curl -i http://localhost:3000/limit=2`

Output: ['Fixed', 'Movable']


##### Creating Dynamic Routes

```javascript
var express = require('express')
var app = express()

var blocks = {
	'Fixed': 'Fastened Securely in position',
	'Movable': 'Capable of being moved',
	'Rotating': 'Moving in circle around its center'
}

app.get('/blocks/:name', function(request, response){  // creates a :name property on request.params
	var description = blocks[request.params.name]
	response.json(description)
})
```

`$ curl -i http://localhost:3000/blocks/Fixed`

Output: 
```
HTTP 1.1 200 OK
Fastened Securely in position
```


Responding for URLs not found.

`$ curl -i http://localhost:3000/blocks/Banana`

Output: 
```
HTTP 1.1 200 OK
Fastened Securely in position
```


## Express Routing

### Intro to concept
Routing refers to determining how an application responds to a client request to a particular endpoint, which is a URI (or path) and a specific HTTP request method (GET, POST, and so on).

Each route can have one or more handler functions, which are executed when the route is matched.

Route definition takes the following structure:
```javascript
app.METHOD(PATH, HANDLER)
```
Where:

- app is an instance of express.
- METHOD is an HTTP request method, in lowercase.
- PATH is a path on the server.
- HANDLER is the function executed when the route is matched.

This chapter assumes that an instance of express named app is created and the server is running. If you are not familiar with creating an app and starting it, see the Hello world example.
The following examples illustrate defining simple routes.

Respond with Hello World! on the homepage:
```javascript
app.get('/', function (req, res) {
  res.send('Hello World!')
})
```
Respond to POST request on the root route (/), the application’s home page:
```javascript
app.post('/', function (req, res) {
  res.send('Got a POST request')
})
```
Respond to a PUT request to the /user route:
```javascript
app.put('/user', function (req, res) {
  res.send('Got a PUT request at /user')
})
```
Respond to a DELETE request to the /user route:
```javascript
app.delete('/user', function (req, res) {
  res.send('Got a DELETE request at /user')
})
```
#### what wikipedia says??
Routing, in a narrower sense of the term, is often contrasted with bridging in its assumption that network addresses are structured and that similar addresses imply proximity within the network. Structured addresses allow a single routing table entry to represent the route to a group of devices. In large networks, structured addressing (routing, in the narrow sense) outperforms unstructured addressing (bridging). Routing has become the dominant form of addressing on the Internet. Bridging is still widely used within localized environments.

#### why's and how's??
#### CRUD
Now that we have an idea of how to collect data from our forms we can start to build some applications! We'll start by building an application that does full CRUD on a resource. Let's first define what we mean by CRUD and resource.

**CRUD** - An *acronym* for *Create*, *Read*, *Update*, and *Delete*. When we talk about building an application with "full CRUD", that means we can perform all of those operations on whatever resources we are working on.

**Resource** - A resource refers to a noun that we are operating on. For example if our resource was "users" and we wanted full CRUD on the "users" resources, we would have routes for the following (these routes are using a convention called RESTful routing):

|   HTTP Verb   |	Path    |	Description    |
|---------------|:----------|------------------|
|GET            |`/users`     |	Show all users |
|GET            |`/users/new` |Show a form for creating a new user|
|GET            |`/users/:id` |	Show a single user |
|GET            |`/users/:id/edit`|Show a form for editing a user|
|POST	        |`/users`     |Create a user when a form is submitted|
|PATCH          |`/users/:id` |Edit a user when a form is submitted|
|DELETE	        |`/users/:id`	|Delete a user when a form is submitted|

#### `methodOverride`
You may have noticed that in the last two routes in the above table, we are using different HTTP verbs! These verbs are `PATCH` and `DELETE` and serve the purpose of updating a resource and deleting a resource.

The problem is that HTML forms only know the GET and POST methods, so we need to override these methods and change them to be PATCH and DELETE for certain requests. This can either be done using a header in the request, or through the query string. To make this process easier, we will use a module called `method-override`.
```
npm install --save method-override
```
```js
// include the method override module
var methodOverride = require("method-override");
// use the method override module and specify the name of the key in the query string which we will assing a value of an HTTP verb to override the POST method (_method=DELETE or _method=PATCH)
app.use(methodOverride("_method"));
```
To use method override, we simply attach the key of `_method` and value of an HTTP verb like `PATCH` or `DELETE` as the value into the query string. You **must** make sure that your method on the form is `POST` as well, even though we will be overriding it. Here is what a form might look like.
```
form(action=`/create-new-user/${user.id}?_method=PATCH` method="POST")
    input(type="text" name="first" value=`${first}`)
    input(type="text" name="last" value=`${last}`)
```
#### CRUD on users
Now that we have an idea of how to build our routes, let's create a simple CRUD application for our users resource. Let's get started with a simple express app in the terminal.
```
mkdir users_crud && cd users_crud
touch app.js
npm init -y
npm install --save express pug body-parser method-override
mkdir views
touch views/{base,index,new,edit,show}.pug # create 5 pug files in the views folder
echo node_modules > .gitignore
git init
git add .
git commit -m "initial commit"
```
Now that we have that folder structure, let's add the following to our app.js
```js
var express = require("express");
var app = express();
var methodOverride = require("method-override");
var bodyParser = require("body-parser");

app.set("view engine", "pug");
app.use(express.static(__dirname + "/public"));
app.use(bodyParser.urlencoded({extended:true}));
app.use(methodOverride("_method"));

var users = []; // this would ideally be a database, but we'll start with something simple to store our users
var id = 1; // this will help us identify unique users for finding, editing, and deleting

app.get("/", function(req, res, next){
  res.redirect("/users");
});

app.get('/users', function(req, res, next){
  res.render('index', {users}); // {users:} is ES2015 object shorthand for {users:users}
});

app.get('/users/new', function(req, res, next){
  // render a page called new.pug inside the views folder. In this file there will be a form that when submitted will POST to /users
  res.render('new');
});

app.get('/users/:id', function(req, res, next){
  // find a user by their id using the find method on arrays. We also have to make sure that we convert req.params.id into a number from a string so we can safely compare
  var user = users.find(val => val.id === Number(req.params.id));
  // render the show page passing in the user we found
  res.render('edit', {user});
});

app.get('/users/:id/edit', function(req, res, next){
  // find a user by their id using the find method on arrays. We also have to make sure that we convert req.params.id into a number from a string so we can safely compare
  var user = users.find(val => val.id === Number(req.params.id));
  // render the edit page passing in the user we found
  res.render('edit', {user});
});

// this route will be requested when a form for creating a user is submitted
app.post('/users', function(req, res, next){
  // add an object to the users array
  users.push({
    // with the key of name and the value of the data from the form
    name: req.body.name
    // and a key of id and a value of id (the variable we are using as a counter)
    id,
  });
  // increment the id for the next user to be added
  id++;
  res.redirect('/users')
});

// this route will be requested when a form updating a user is submitted
app.patch('/users/:id', function(req, res, next){
  // find the user
  var user = users.find(val => val.id === Number(req.params.id));
  // update their name with the data from the form
  user.name = req.body.name;
  res.redirect('/users');
});

// this route will be requested when a form for deleting a user is submitted
app.delete('/users/:id', function(req, res, next){
  // instead of finding the entire user, let's just see what index they are at in our users array
  var userIndex = users.findIndex(val => val.id === Number(req.params.id));
  // then we can remove one item from the users array starting at the correct index we found above
  users.splice(userIndex,1);
  res.redirect('/users');
});

app.listen(3000, function(){
  console.log("Server is listening on port 3000");
});
```
You may also notice that we are starting to place a `next` parameter in our callback functions. We will see in a later section why this is so helpful, but in a nutshell it is useful for handling errors and making sure they are propagated to the top.