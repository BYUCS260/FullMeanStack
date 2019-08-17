# FullStack
Hints for creating an application from scratch with Vue/Node/Mongo

In this tutorial you will create an application that will implement comments for a blog. You will allow the users to create comments and then upvote other people's comments. The following steps will use express to set up your project and then build the front and back end in the project.

First create an express project
```
express comment
cd comment
npm install
npm start
```
This will start a web server on port 3000. Take a look at the file in bin/www which is a node.js app that is run with "npm start". Files in the "public" directory will be served by the node server.
Lets get started with a simple Vue application "index.html" inside of the "public" directory.
```
<!DOCTYPE html>
<html>

<head>
  <link rel="stylesheet" type="text/css" href="stylesheets/style.css">
  <meta charset="utf-8">
  <title>Comments</title>
</head>

<body>
  <div id="app">
    {{test}}
  </div>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.18.0/axios.min.js" integrity="sha256-mpnrJ5DpEZZkwkE1ZgkEQQJW/46CSEh/STrZKOB/qoM=" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.2/dist/vue.js"></script>
  <script src="javascripts/app.js"></script>
</body>
</html>
```
Now create the javascript in "public/javascripts/app.js".
```
/*global axios */
/*global Vue */
var app = new Vue({
  el: '#app',
  data: {
    test: "Hello World",
  },
  created: function() {
  },
  methods: {
  }
});
```
You now have your hello world Vue app.
Now we are going to modify the javascript to include some new data with comments.
```
  data: {
    comments : [
      'Comment 1',
      'Comment 2',
      'Comment 3',
      'Comment 4',
      'Comment 5'
    ],
    test: "Hello World",
  },
```
And add some Vue code to display the comments.
```
        <ul>
            <li v-for="item in comments">
                {{ item }}
            </li>
        </ul>
```
Now lets add upvotes to our comments and make each array element an object.
```
    comments : [
      {title:'Comment 1', upvotes:5},
      {title:'Comment 2', upvotes:6},
      {title:'Comment 3', upvotes:1},
      {title:'Comment 4', upvotes:4},
      {title:'Comment 5', upvotes:3}
    ],
```
And change the view to show upvotes.
```
            <li v-for="item in comments">
                {{item.title}} - upvotes: {{item.upvotes}}
            </li>
```
Of course, you want to sort the comments based on popularity, so include a computed property "sortedComments".
```
            <li v-for="item in sortedComments">
                {{item.title}} - upvotes: {{item.upvotes}}
            </li>
```
And specify the javascript sort function in your computed property.
```
    computed: {
        sortedComments() {
            return this.comments.sort((a, b) => {
                var rval = 0;
                if(a.upvotes > b.upvotes) {
                    rval = 1;
                } else if(a.upvotes < b.upvotes) {
                    rval = -1;
                }
                return(rval);
            })
        }

    },
```
Test your application to make sure the comments are sorted by upvotes.  Now that we have the list displayed, it would be nice to add comments to the list. First create a function that will add an object to the comments array.
```
    methods: {
        addComment() {
            this.comments.push({ title: this.newComment, upvotes: 0 });
            this.newComment = "";
        },
    }
```
Then add a form to call the function in your html file.
```
        <form v-on:submit.prevent="addComment">
            <input type="text" v-model="newComment">
            <button type="submit">Add</button>
        </form>
```
Make sure everything is working. You should see the new comment in your list.

Now that we have the ability to add comments, we ought to allow the user to upvote the comments he likes. Next to each comment, we will place a click-able character that the user can select to increment the upvotes. Notice that the parameter to incrementUpvotes is passed by reference so the list is automatically rearranged.
First, modify the html to have the clickable character

```
            <li v-for="item in sortedComments">
                <span v-on:click="incrementUpvotes(item)">^</span>{{item.title}} - upvotes: {{item.upvotes}}
            </li>
```
Then add the function to app.js
```
        incrementUpvotes(item){
            item.upvotes = item.upvotes+1;
        }
```
The click in the view called the controller which changed the model which then updated the order in the view.
Now lets make things look a little better. Use the bootstrap css to spruce things up.
```
<!DOCTYPE html>
<html>

<head>
    <link rel="stylesheet" type="text/css" href="stylesheets/style.css">
    <link href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css" rel="stylesheet">
    <meta charset="utf-8">
    <title>Comments</title>
</head>

<body>
    <div id="app">
        <form v-on:submit.prevent="addComment">
            <input type="text" v-model="newComment">
            <button type="submit">Add</button>
        </form>
        <ul>
            <li v-for="item in sortedComments">
                <span class="glyphicon glyphicon-thumbs-up" v-on:click="incrementUpvotes(item)"></span> {{item.title}} - upvotes: {{item.upvotes}}
            </li>
        </ul>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.18.0/axios.min.js" integrity="sha256-mpnrJ5DpEZZkwkE1ZgkEQQJW/46CSEh/STrZKOB/qoM=" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.2/dist/vue.js"></script>
    <script src="javascripts/app.js"></script>
</body>

</html>
```
Test the front end to make sure everything is working so we can attach the back end.
If you are stuck, you might want to refer to my working front end index.html and app.js

Now we will install Mongoose which will provide schemas on top of mongodb.
```
npm install --save mongoose
```
The --save flag updates the packages.json file with mongoose so you can easily restore them with a "npm install" command.
You may get an error saying something like

lib/kerberos.h:5:27: fatal error: gssapi/gssapi.h: No such file or directory
This is a good time to figure out how to install packages. If you google for this error, you will find that the kerberos library is not installed. To fix this use:
```
sudo apt-get install libkrb5-dev
```
Then rebuild your npm modules
```
sudo npm rebuild
```
Lets look at the express project structure
The node project created by express has the following directory structure:
app.js - This file is the launching point for our app. We use it to import all other server files including modules, configure routes, open database connections, and just about anything else we can think of.
bin/ - This directory is used to contain useful executable scripts. By default it contains one called www . A quick peak inside reveals that this script actually includes app.js and when invoked, starts our Node.js server.
node_modules - This directory is home to all external modules used in the project. As mentioned earlier, these modules are usually installed using npm install . You will most likely not have to touch anything here.
package.json - This file defines a JSON object that contains various properties of our project including things such as name and version number. It can also defined what versions of Node are required and what modules our project depends on. A list of possible options can be found in the npm documentation.
public/ - As the name alludes to, anything in this folder will be made publicly available by the server. This is where we are going to store JavaScript, CSS, images, and templates we want the client to use.
routes/ - This directory houses our Node controllers and is usually where most of the backend code will be stored.
views/ - If we were not using Angular, we could generate interactive views here.
In addition to the above files structure, we are going to add one more folder. Create a new folder called "models":
```
mkdir models
```
This folder will contain our Mongoose schema definitions.
Now we are going to set up the mongo database for the node.js backend. We will use mongoose to set up the schema. Chapter 16 of the book discusses mongoose in detail. Add the following code to the top of your app.js file [right after require('body-parser')] to connect to the mongod. Make sure that mongod is running on your instance.
```
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/comments');
```
Now create the file "Comments.js" in the models directory with the following content.
```
var mongoose = require('mongoose');
var CommentSchema = new mongoose.Schema({
  title: String,
  upvotes: {type: Number, default: 0},
});
mongoose.model('Comment', CommentSchema);
```
Now add the model to your app.js file right after the mongoose.connect call.
```
require('./models/Comments');
```
Now we need to open up REST routes to the database. We want the user to be able to perform the following tasks:
```
view comments
add a comment
upvote a comment
```
These tasks correspond to the following routes:
```
GET /comments - return a list of comments
POST /comments - create a new comment
PUT /comments/:id/upvote - upvote a comment, notice we use the comment ID in the URL
```
Lets start be opening up a GET route in routes/index.js
```
var mongoose = require('mongoose');
var Comment = mongoose.model('Comment');

router.get('/comments', function(req, res, next) {
  Comment.find(function(err, comments){
    if(err){ return next(err); }
    res.json(comments);
  });
});
```
Notice that the Comment variable refers to the Comment model we defined earlier.
Before we can test that the route works, we need data in the mongo database, so lets create a POST route in routes/index.js

```
router.post('/comments', function(req, res, next) {
  var comment = new Comment(req.body);
  comment.save(function(err, comment){
    if(err){ return next(err); }
    res.json(comment);
  });
});
```
Notice that we created a Comment object from the req.body and saved it to the mongo database using the mongoose connection to the mongo database.
Now lets test the routes using curl from your instance. First type cntrl-C to kill your server, then "npm start" to start it up again.
```
curl --data 'title=test' http://localhost:3000/comments
```
This should return something like this:
```
$ curl --data 'title=test' http://localhost:3000/comments
{"__v":0,"title":"test","_id":"563ba5ac1a761cf149c0b258","upvotes":0}
```
Now lets test the GET route.
```
curl http://localhost:3000/comments
```
Should return something like this
```
[{"_id":"563ae37a13190ba93fc96a34","title":"test","__v":0,"upvotes":1}]
```
You can also access the GET route through the URL in your browser. Test it to make sure everything is working.
Notice that the upvote REST interface requires us to find a particular comment before operating on it. In order to make this easier, we can create a route to preload a comment object in routes/index.js using the express param function.
```
router.param('comment', function(req, res, next, id) {
  var query = Comment.findById(id);
  query.exec(function (err, comment){
    if (err) { return next(err); }
    if (!comment) { return next(new Error("can't find comment")); }
    req.comment = comment;
    return next();
  });
});
```
Now, whenever we create a route with :comment in it, this function will be run first to get the comment out of the database. The router.param allowed us to define this middleware that is passed to the route. We use the query interface for mongoose to simplify the access.
Lets use this middleware function to create a route for returning a single comment

```
router.get('/comments/:comment', function(req, res) {
  res.json(req.comment);
});
```
Since the :comment part of the route was interpreted by the middleware that put the result from mongoose into the req.comment object, all we have to do is to return the JSON back to the client.
The :comment part of the URL will be the ID given to the comment in mongo. So, you should be able to enter a URL like this to test your setup:

```
http://YOURIP/comments/54f4b19425b53f6a052851ce
```
Now let's implement the route to allow upvoting. We will use our middleware to identify the comment and then open up a route on this comment to upvote it. Add the upvote method to the models/Comments.js schema.
```
CommentSchema.methods.upvote = function(cb) {
  this.upvotes += 1;
  this.save(cb);
};
```
Then create a PUT route in routes/index.js
```
router.put('/comments/:comment/upvote', function(req, res, next) {
  req.comment.upvote(function(err, comment){
    if (err) { return next(err); }
    res.json(comment);
  });
});
```
You should now be able to test your route using curl. First access the route to GET all of the comments. Find the id of one of the comments, and use curl to upvote it.
```
curl -X PUT http://localhost:3000/comments/<COMMENT ID>/upvote
```
Now use the URL to make sure that the upvote count was incremented.
Now that our backend is working, we just need to wire it up to our vue frontend. First we will create a getAll() function to retrieve comments from our REST service in public/javascripts/app.js.
```
        async getall() {
            console.log("get all");
            var url = "http://yourserver:4200/comments"; // This is the route we set up in index.js
            try {
                let response = await axios.get(url);
                this.comments = response.data; // Assign array to returned response
                console.log(this.comments);
                return true;
            }
            catch (error) {
                console.log(error);
            }
        },
```

Upon success, we will copy the data from the GET REST service into our this.comments array. Vue will update the displayed html. 
You will want to call getall() when the app is created with
```
    created: function() {
        this.getall();
    },
```
Now that you have implemented one backend interface, the others should be easy. Lets modify the addComment function to write the output to the mongo database.
```
        addComment() {
            var url = "http://clementbyu.com:4200/comments";
            axios.post(url, {
                    title: this.newComment,
                    upvotes: 0
                })
                .then(response => {
                    console.log("Post Response "); 
                    console.log(response.data);
                    this.comments.push(response.data);
                })
                .catch(e => {
                    console.log(e);
                });
            console.log(this.comments);
            this.newComment = "";
        },
```
When the call to the POST /comments REST service is successful, the object with the _id field will be returned in response.data. In the .then block, we can push this complete object onto the array so that the upvote will know the _id to send to the back end.

Test this function to make sure you can create new comments and see them displayed. You should be able to refresh the page and still see them.

Now you need to be able to upvote your comments. Follow the same process of adding an axios call your PUT REST route. We want to avoid race conditions between different browsers accessing the same comment, so we will replace our view of the number of upvotes with the response data. 
```
        incrementUpvotes(item) {
            var url = "http://clementbyu.com:4200/comments/"+item._id+"/upvote";
            axios.put(url)
                .then(response => {
                    console.log(response.data.upvotes);
                    item.upvotes = response.data.upvotes;
                })
                .catch(e => {
                    console.log(e);
                });
            console.log("URL "+url);
        }
```
Test to make sure the upvotes are maintained across refreshes.
