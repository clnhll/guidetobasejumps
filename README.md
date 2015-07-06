#Intro to Yeoman Angular-Fullstack Basejumps

>Hey FreeCodeCampers! This guide is here to help you navigate creating your first basejump. When I encountered it, I had no idea what was going on and spent weeks learning all of these things myself. Everything here is stuff I wish I had known coming into the basejumps. —<a href="http://twitter.com/clnhll">@clnhll</a>

Yeoman is a tool that allows you to generate barebones apps based on different software stacks using “generator” packages made by developers who want to make your life easier. These packages streamline your time developing and deploying websites using your platform of choice. We’re using a full-stack MEAN (MongoDB, ExpressJS, AngularJS, NodeJS) generator called generator-angular-fullstack by DaftMonk (https://github.com/DaftMonk/generator-angular-fullstack).  

Once you’ve completed the <a href="http://www.freecodecamp.com/challenges/Waypoint:%20Get%20Set%20for%20Basejumps">Waypoint: Get Set for Basejumps</a>, use this guide to navigate the base structure of your new app and learn how to interact with the database as well as the user.  

###Throughout this guide:  
__bolded/names/with.extensions_</a> are directories and files in the project file structure  
<a>highlighted.items/are/hypothetical</a> URLs that allow access to different pages in your app  
*italicizedItems* are function and object names within your code

##Part 1: Frontend
First things first: All your user-facing files and angular files are in **/client/app/**  

1. **app.js**: defines your app and includes some basic app-wide functions, you probably don’t really need to mess with it unless you’re trying to add more dependencies to your app. We’re not gonna worry about that right now.  
2. **app.css**: an app-wide stylesheet, you can put styles here if you want but I’d recommend you put them in **main/main.css**, as these styles are also app-wide.  
3. **main/**: this folder contains what the user sees first when they load up your site. **main.html** is the page template, **main.js** routes the user to **main.html** when the user goes to the top level directory of your website—that is, <a>http://yourapp.wherever.itis/</a> with no <a>/other/url/hierarchy</a>. You’ll also learn soon that you can define your app’s <a>/url/heirarchy/fairly/arbitrarily</a>. You won’t really need to edit **main.js** or **main.controller.spec.js**, so let's not worry about those right now. If you look through the **main.html** file you’ll see it uses *ng-repeat* to show *things* in *awesomeThings*. Where does it get *awesomeThings*?  
4. **main/main.controller.js**: all of the javascript functions you want to use to interact directly with the user go here! You’ll put functions here to add and remove items from your database, refresh views for your user, etc. Here, *awesomeThings* are pulled from your database and added to the local scope so your HTML view can display them! How cool! We’ll get to adding custom objects to your database in a minute.

Great! Now you know how to interact with the user! But what if you want your app to have another page that does something else? Maybe **main.html** shows the home page, but you want a page that shows a form to add a poll? maybe <a>http://yourapp.wherever.itis/newpage</a>? This is where the yeoman generator comes in handy.
###Creating a new route

	>> yo angular-fullstack:route newpage 
   Typing the above into your command-line will generate a **newpage/** route for your app! It automatically generates all the necessary files within your **/client/app/newpage** folder, like your **/client/app/main** folder, with a **newpage.controller.js**, **newpage.controller.spec.js**, **newpage.js**, and **newpage.html**. These all pretty much behave like the ones in the **main/** route. If you’re accessing the database in your newpage controller, you’ll want to add *$http* to the list of dependencies in **newpage.controller.js** the same way it’s included in **main.controller.js**. Check out **main.controller.js** and see if you can figure it out.  

###Creating a new directive
Do you remember custom directives from the shaping up with angular course? you can also make a custom directive!

	>> yo angular-fullstack:directive newdirective

And if you need an html template for your custom directive (maybe you’re just making a directive to clean up your HTML code), tell it to make an html file when it prompts you to.

Okay! But what about the database? What’s going on there?

##Part 2: Backend
Your app’s backend api that interacts with your database is located in **/server/api**  
Let’s take a look at **/server/api/thing**: 

1. **index.js**:  this file routes the $http API requests made from your app’s front-end to the appropriate function in **thing.controller.js ** 
2. **thing.controller.js**: Here is where we actually deal with the database! Take a minute to look through here and figure out what’s going on. These functions will: return all items in a collection, return a single item from a collection when passed its id, post an item to a collection, update an item in the collection (this doesn’t really work as intended out of the box, you may need to be creative and rewrite that one), and of course, delete an item from the collection.  
3. **thing.model.js**: Here, the actual structure of a *thing* object is defined. You can add or remove any fields you want from the *thing* model, and as long as they’re syntactically correct they won’t break anything, even if there are *things* with different schemas in your database already. But! You don’t just have to edit the *thing* model to make a new type of collection, because generator-angular-fullstack can do it for you!

###Creating a new API endpoint:  

	>> yo angular-fullstack:endpoint whatsit
 Using this line, you get Yeoman to automatically generate another API endpoint and new kind of collection for your database. Now we have things as well as whatsits! Feel free to open up the **whatsit.model.js** and define your whatsit however you like.

##Part 3: Interfacing Between Frontend & Backend
###Accessing the database from your frontend:  
You must have noticed in **main.controller.js** how *things* were retrieved from the database and displayed:
  
~~~javascript
 $http.get('/api/things').success(function(awesomeThings){  
	$scope.awesomeThings = awesomeThings;  
});

~~~  

What this does is call the api with a “get” request, which is then routed by **/server/api/thing/index.js** to the *exports.index* function in **thing.controller.js**. You’ll also notice in **main.controller.js** that there are included examples of *$http.post* and *$http.delete* functions too! How nice!  

###Quick tip: 
Say you want something to show up on the user view when you add it to the database. A new *thing* object will show up in an *ng-repeat* loop on your HTML view if you simply add it to your local array with  

~~~javascript

$scope.awesomeThings.push(newThing);
~~~

But you’ll still need to add it to your database collection. Add it to your collection with  

~~~javascript

$http.post('/api/things', newThing);	

~~~
But wait! You’ll soon realize that while all the other things in your *$scope.awesomeThings* array have unique ids assigned by MongoDB (under the *thing.\_id* property), your *newThing* object will not, which will make it hard for you at some point to make database actions on it (deleting it from your database requires you to use its *._id* property). So what you’ll want to do after you add it to your *$scope.awesomeThings* array (because we want it to show up on the user’s page immediately), is to call  

~~~javascript
$http.post('/api/things', newThing).success(function() {
	$http.get('/api/things').success(function(awesomeThings) {
		$scope.awesomeThings = awesomeThings;
	}
};
~~~
to post it to your database and replace the local array with a new array that has the database’s version of your *newThing* object, unique *._id* and all. Notice the *$http.get* call is only called once the *$http.post* call has succeeded! This is because *$http* calls are asynchronous, so we want to ensure that your program waits to call *$http.get* and refresh your local array until it’s sure your *newThing* has been posted to the database.

##Part 4: Dynamic URLs using $routeParams, more useful APIs
What if you have a lot of users posting *things* to your website? Maybe your users want to have a profile, or a wall, of the *things* they’ve posted, and they want to be able to share it with their friends with a url? You can do that, no biggie!

Let’s say you used 

	>> yo angular-fullstack:route wall

to generate a <a>http://myapp.wherever.com/wall/</a> route for your users. You want a link to <a>http://myapp.wherever.com/wall/someUsername</a> to show a specific user’s *things*.  
Browse to **/client/app/wall/wall.js** and notice that it detects what URL the user is requesting before acting on it:  

~~~javascript
$routeProvider.when('/wall', …
~~~
You can customize that path to catch when a user is trying to see a profile associated with a specific username like so:  

~~~javascript
$routeProvider.when('/wall/:username', …
~~~
The colon before "username" indicates that this is a variable, which is then passed to the *$routeParams* module. In **wall.controller.js**, include *$routeParams*:  

~~~javascript
.controller('WallCtrl', function ($scope, $routeParams) { …
~~~
Then later on in **wall.controller.js**, you can see what username was requested in the URL by referring to the variable generated by *$routeProvider* using something like  

~~~javascript
var wallOwner = $routeParams.username;
~~~
There are two more things you have to do before this to be useful to you, however. Say you want to show all the *things* associated with the username requested with that page: you must first  
 
1. Have a “username” or “owner” field in your *thing* schema at **/server/api/thing/thing.model.js**  
2. Write a custom route in **/server/api/thing/index.js** to catch a request for a specific username. The request from your frontend might look something like:

~~~javascript
 $http.get('/api/things/' + username).success( … 

~~~
so you’ll add a line into your **index.js** like:

```javascript

router.get('/:user', controller.indexUser);
```  
  and then in **thing.controller.js** you’ll write an *exports.indexUser* function like so:

~~~javascript
exports.indexUser = function(req, res) {
	Thing.find({owner:req.params.user}, function (err, things) {
		if(err) return res.send(500, err);
		res.json(200, things);
	});
};
~~~


Warning!!! this method only works right if usernames are absolutely unique between users. The default authentication system that comes with the angular-fullstack generator does not have unique usernames, so you’re probably better off using the *user._id* field to determine unique users in your database for now, unless you want to implement unique user names yourself by altering your **/api/user/user.model.js**, **/api/user/user.controller.js**, and your **/app/client/account/signup/signup.controller.js**. Thankfully, you should know how to go about doing all that after reading this guide!

##Part 5: Auth, isLoggedInAsync()
You may have noticed if you opened up **/client/app/admin/admin.controller.js** that it calls the *Auth* module like so:

~~~javascript
.controller('AdminCtrl', function ($scope, $http, Auth …
~~~
It’s pretty useful to have *Auth* available in your controller to detect if a user is logged in, or to get information about the current user. In the body of your controller you can add 

~~~javascript
$scope.getCurrentUser = Auth.getCurrentUser;
$scope.isLoggedIn = Auth.isLoggedIn;
~~~
And then you can use *isLoggedIn()* or *getCurrentUser()* in the HTML view for your controller!  
But wait—what if you’re trying to decide if a user is logged in before you make an *$http* call? It’s not guaranteed that it will work because *isLoggedIn()* is actually an async call. If you want to force something to wait until after *isLoggedIn()* is successful before it gets called, you should include *Auth.isLoggedInAsync*:

~~~javascript
$scope.isLoggedInAsync = Auth.isLoggedInAsync;
~~~	

*isLoggedInAsync* takes a callback function as an input, and passes the callback function a *true* boolean if the user is logged in, and a *false* if the user is not, so you can call it like so: 

~~~javascript
$scope.isLoggedInAsync(callback(bool) {
	if (bool) { // do thing if they’re logged in } 
	else { // do different thing if they’re not logged in }
});
~~~

##Bonus: SocketIO 
If you've gotten to the Stock Charting basejump you may have noticed that the bonus criteria is to have your stock list live update across clients. This can be accomplished with SocketIO, but that’s not all SocketIO can do. Remember earlier, I mentioned that you had to put *$http.get()* requests within the *success* callback of an *$http.post()* function when adding things to your database to get them to play nicely due to async? SocketIO keeps a user’s browser environment synced with your database in realtime. This has two practical upshots:  

1. You no longer have to run nested *$http* calls to update your local data with database data; it is all managed automatically
2. You can push database changes live to users on different machines all at the same time  

Even better, if you just include SocketIO when prompted during the yeoman angular-fullstack setup, there is absolutely no work involved to include it. It works out of the box, has a working demo on the **main/** route, and you can learn how to use it yourself by simply looking at how they include it in **main.controller.js** (so I won’t go any further into detail).

##Epilogue
If you have any issues not covered in this guide: 

1. google google google google duckduckgo
2. bug @freecodecamp and @clnhll on twitter
3. did you miss a semicolon? a comma? 
4. make a big loud stink in the freecodecamp help gitter.

I believe in you!