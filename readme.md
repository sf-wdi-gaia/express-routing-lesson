# Intro to express and express-routing

### Objectives
*After this lesson, students will be able to:*

- Use and configure middleware
  - We'll use body-parser to parse form submissions as JSON
- Write out the skeleton of a RESTful API
- Interact with HTTP verbs using CURL or an app
- Identify the HTTP verbs we'll be using for an API

### Preparation
*Before this lesson, students should already be able to:*

- Explain HTTP requests/responses
- Explain MVC
- Write and explain basic javascript

## Recapping Node and Intro to Express - Intro (5 mins)

#### First let's review

* What is Node?

Node is a low-level, non-blocking, event-driven platform which allows you to write JavaScript on the server-side.

* What is npm?

npm is Node's package manager. It's used to manage dependencies. Think of it like RubyGems.

* What is express.js?

Express.js is a simple web framework for Node.js. It provides many features for you to start using right away (Routing, Sessions) that you would have to do yourself if using vanilla Node.


## Templates in Express 

Express comes with a default templating engine called [jade](http://jade-lang.com), a high performance template engine heavily influenced by [HAML](http://haml.info).  Like HAML, jade simplifies writing html by eliminating the need for parts of html tags and utilizing white space.  

You may explore jade and haml on your own, but we'll be using another common templating engine called [EJS](http://www.embeddedjs.com/) (Embedded JavaScript) because when we get to Sinatra and Rails, we will use something called Embedded Ruby or ERB, which is very similar.

Instead of sending some text when we hit our site let's have it serve an index page.

#### Install ejs
```
npm install ejs --save
```
You can uninstall from a project with:

```
npm uninstall ejs --save
```

#### Setting up ejs to render the index

To change your rendering engine you'll need to edit your apps configuration in `app.js`. We also have to change what happens when a user GETs '/'. Let's get it to render our index template instead of sending 'Hello World'.

```javascript
// app.js

var express = require('express');
var app     = express();
var port    = process.env.PORT || 3000;

app.set('views', './views');
app.set('view engine', 'ejs');  

app.get('/', function(req, res) {
    res.render('index');
});

app.listen(port);
console.log('Server started on ' + port);
```

Notice that we have added:

```javascript
res.render('index');
```

#### Creating views in Express

How about an ejs index page:

```bash
touch views/index.ejs
```

And add this code:

```
  <!doctype html>
  <html lang="en">
  <head>
    <title>Welcome to express.js!></title>
  </head>

  <body>
    <h1>Express and EJS</h1>
    <div class="container">
      <p> This is a paragraph of text. Yay! </p>
    </div>
  </body>
```


## Adding Routes to our app 

Let's add some routes. This should all be familiar but let's go through it.

[ExpressJS 4.0](https://scotch.io/tutorials/learn-to-use-the-new-router-in-expressjs-4) comes with the new Router. Router is like a mini Express application. It doesn’t bring in views or settings but provides us with the routing APIs like `.use`, `.get`, `.param`, and `route`.

First we define our _router_. This is what handles our routing. It's normally better to use this way of doing routes (and extracting them in to their own files) as it makes applications more modular, and you won't have a 500 line app.js.

```javascript
var express = require('express');
var app     = express();
var port    = process.env.PORT || 3000;
var router  = express.Router();
```

This needs to be under the definition of `var app`!  Then we add our routes.

```javascript
router.get('/', function(req, res) {
  res.render('index', { header: 'index!'});
});

router.get('/contact', function(req, res) {
  res.render('contact', { header: 'contact!'});
});

router.get('/about', function(req, res) {
  res.render('about', { header: 'about!'});
});
```

At the bottom of the page add:

```javascript
app.use('/', router);
```

As we saw before we are rendering our template and then passing in a local variable (_header_) to use in our template, just like instance variables defined in our controller.

## Restful Routing - Intro (10 mins)

We are going to use the RESTful standard to build our web apps. REST stands for REpresentational State Transfer and is an organizational standard for web architecture designed "to induce performance, scalability, simplicity, modifiability, visibility, portability, and reliability," in the words of its author, Roy Thomas Fielding.

So far, we've covered how to handle GET requests, but we can create callbacks for all types of requests. For example, if we want to create a restful controller for the resource cars, it would look like this:

```javascript

var carRouter = express.Router();

carRouter.get('/', function(req, res) {
  // INDEX
});

carRouter.get('/:id', function(req, res) {
  // SHOW
});

carRouter.get('/new', function(req, res) {
  // NEW
});

carRouter.post('/', function(req, res) {
  // CREATE
});

carRouter.get('/:id/edit', function(req, res) {
  // SHOW
});

carRouter.put('/:id', function(req, res) {
  // UPDATE
});

carRouter.delete('/:id', function(req, res) {
  // DELETE
});


app.use("/cars", carRouter)
```

We've defined that the endpoint for the car resource will be "/cars".
So the code above will create these 7 routes:

```javascript

GET    /cars
GET    /cars/:id
GET    /cars/new
POST   /cars
GET    /cars/:id/edit
PUT    /cars/:id
DELETE /cars/:id

```

## BodyParser and handling params/JSON

When data is sent to the server via a POST request (from a form, for example), the content of the request is passed as a string, but we want to access it as if it was a JSON object:

If we have a form like this:

```html
    <form>
      <input type="text" name="car[make]">
      <input type="text" name="car[model]">
    </form>
```

Once this form is submitted, by default, the data on the server will look like this:

```json
{
  "car[make]"  : "value",
  "car[model]" : "value"
}
```

...but this is not really convenient, as accessing the data will be a bit complex to parse:

```javascript
req.body['car[make]']
```

It would be a bit easier if we could use the data like:

```javascript
req.body.car.make
```

For this we will use `body-parser`.

## Configure your app to use body-parser

First add the package to your `package.json` dependencies:

```json
"body-parser": "^1.13.2"
```

Now in `app.js`, add:

```javascript
app.use(bodyParser.urlencoded({ extended: false }));

```

The params passed with a request will be "decoded" automatically, allowing you to use dot notation when working with JavaScript objects.

If you are writing an API, meant to receive and send JSON, you would change the line above to:

```javascript
app.use(bodyParser.json());
```

Now the app will decode all JSON received in the body of a client request.


## Independent Practice

> ***Note:*** _This can be a pair programming activity or done independently._

In the same file, try to create the 7 Restful Routes for the resource "car". Every method should return some text saying the HTTP Verb, which URI has been used to do the request and which REST action it corresponds to.

Example, for a POST request to `/cars` the text sent back should be:

```
POST request to /cars, this is the CREATE action
```

Also, test your application with `cURL` requests to each of the RESTful endpoints.

## Conclusion
A framework can be overwhelming at the start, after a couple of days you will see how it makes your life easier.  We will work more on how to make RESTful controllers, this is just an introduction.

- What is Middleware and why is it helpful in Express?
- Explain how body-parser helps decode information in your application.
- Identify some similarities and differences between Express and Sinatra.


## Licensing
All content is licensed under a CC­BY­NC­SA 4.0 license.
All software code is licensed under GNU GPLv3. For commercial use or alternative licensing, please contact legal@ga.co.