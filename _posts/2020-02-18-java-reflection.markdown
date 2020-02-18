---
layout: post
title: "Java: Towards a Rails-Like Router in a Home Grown Java HTTP Server"
date:   2020-02-23 12:00:00 -0400
categories: coding
---

I recently wrapped up a humble, but functional, [HTTP server in
Java](https://github.com/breadoliveoilsalt/http-server-java). Much of my
architecture was based off what I know about the Ruby on Rails model, where a
`routes.rb` file is responsible for drawing a link between a path requested by a
client and a particular controller.  For example, as the [Rails
Guides](https://guides.rubyonrails.org/routing.html#resources-on-the-web)
describe, configuring your `routes.rb` file with a simple line like so...

```Ruby
# config/routes.rb

  resources :photos
```

...maps a host of potential client requests specifying a `/photos` path to a
PhotosController, a class in its own right.  The instance methods of the
PhotosController know how to respond to the request depending, in large part,
on the HTTP method from the client's request. You can read about all of the amazing consequences of that one
line above
[here](https://guides.rubyonrails.org/routing.html#resources-on-the-web).
Suffice it to say, this mapping, through such a simple declaration, is very
sophisticated and contributes to the magical mystique of Rails.

Inspired by the Rails mode, my HTTP server has a Router class, an instance
which effectively effectively serves as a `routes.rb` file.  When starting this
project, I knew that I wasn't in a spot to pull off a mapping of requested
paths to controllers as elegantly as Rails does.  And originally, I started
hard coding my Router so it would assign a controller to a request if
a particular path requested included particular HTTP method.  In
pseudocode, something along the lines of:

```
if path == "/photos" && HTTPmethod == "GET", look to PhotosController#get
if path == "/photos" && HTTPmethod == "OPTIONS", look to PhotosController#options
if path == "/photos" && HTTPmethod == "POST", look to PhotosController#post
if path == /comments && HTTPmethod == "OPTIONS", look to CommentsController#options
if path == /comments && HTTPmethod == "POST", look to CommentsController#post
etc...ad nauseum...
```

Of course, such hard coding was very fragile, brittle, and unresiliant to change.  I
began to wonder if there was a better way. Could my Router simply point a requested path to a particular controller,
moving closer to the simplicity of a Rails router? Something along the lines of:

```
if path == "/photos" => PhotosController
if path == "/comments" => CommentsController
```

I began poking around Java's reflection library in a quest for something along
those lines, and below is a high level description of what I came up with. The
complete code can be found [here]().

There are shortcomings with this approach.  For example, right now, my router is not
sophisticated enough to handle nested routes.  And there's one Java unchecked
type warning that I can't quite fix due to the use of reflection.  But it's a
start towards a more Rails-like router in Java.  And it was really cool to learn
about Java reflection.  So to dive right in...
<p/>
-------------------
-------------------
<p/>
Here's the mission:

* When configuring the server to respond to certain requested paths (a.k.a, resources), I just want to point a path to a controller in a router.  For example, the router tells the rest of the program that if the client requests a `/photos` path, the program should look to a `PhotosController`.
* A controller's own methods will define which HTTP methods are available for the relevant path. Rather than use the Rails conventions for controller methods (`#index`, `#show`, etc.), if I want a controller to respond to a `GET` request, it will have a `get()` method that returns a response.
* Because we are not mapping every possible route to every possible method explicitly in the router, we will need another class to intermediate between the router and a controller.  This class, which I call a ControllerMapper below, will be responsible for calling a controller method that responds to an HTTP request (e.g., calling `get()` in the controller if the request has a HTTP `GET` method).  Alternatively, if there is no controller method corresponding to the HTTP method requested, the ControllerMapper will indicate that a "[405 Method Not Allowed](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/405)" response is in order.

A few assumptions (a.k.a, what actually happened with my server):
* We're not concerned about nested routes.
* There are other mechanisms, such as homegrown middleware, that handle serving up static files or listing the contents of a directory.
* Most of the "business logic" within the server, including the middleware, manipulates a Response object, based on a Request object. The controllers are no exceptions - they mutate the Response object when dealing with it.
* Due to historical reasons, we're using Java 8.

We'll tackle this mission with the help of the [Java reflection library](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html). Long story short, it can do some really cool things to abstract away classes and methods and to play with some metaprogramming. For example, you can call reflection methods to examine all of the possible methods of any given class, to probe what those methods return, to construct an instance of that class, and to call methods on the instance -- all without specifying up front what the concrete class is.  Hopefully I'll do a good job below of explaining how I relied on these techniques for my router and its controllers.

Let's start off simple-ish (below is not my complete code, and any references to a
  PhotoController below are simply an example. My code did not
handle photos). Say we have a Controller
superclass that subclass controllers will implement.  I want to pass to a
controller subclass a Request object (representing a parsed client request) and a
Response object (which will eventually be parsed and sent back to the client).
I also want each controller subclass to be able to reveal which HTTP methods it knows how
to respond to.  Here's a start to the Controller superclass, to kick things off:

```java
public abstract class Controller {

    protected final Request request;
    protected final Response response;

    public Controller(Request request, Response response) {
        this.request = request;
        this.response = response;
    }

    public Set<String> getRecognizedHTTPMethods() {
        /* Something here to come...
           Subclasses will inherit this method to reveal which HTTP methods they
           respond to. */
    }

}
```

Assume we want to have a PhotosController subclass (which will be responsible for handling client requests to a "/photos" path).  Also assume that we want this particular controller to handle requests only if they have the HTTP method `GET` or `POST`.  In accordance with our Rails-like goals, our PhotosController will look something like this:

```java
public class PhotosController extends Controller {

        /* We call super to populate the `request` and `response` fields.  
           These fields are `protected`, so subclass can access them. */
    public PhotosController(Request request, Response response) {
        super(request, response);
    }

          /* Below, returning a Response object is a flag that this method
             handles an HTTP method.  The use of this flag will be shown
             further below. */
    public Response get() {
          /* Here would be some code to talk talk to a database, or get
             a static file, or whatever to manipulate the original Response
             object.  E.g.:  
             this.response = doSomeStuffToTheResponse(); */
       return response;
    }

          /* For a `post()` method below, let's return our response object
             again as a flag that this method can handle an HTTP method. */
    public Response post() {
          /* We need the request object to see what came in from the client.  E.g.:
             grabParams(this.request);
             Then we manipulate the response. E.g.:
             this.response = doSomeOtherStuffToTheResponse(); */
       return response;
    }

}
```

So let's assume now that our PhotosController knows how to respond to a `GET`
and `POST` HTTP method in a client request and will manipulate a Response
object accordingly.  How can the PhotosController tell other parts of our
application (including the router) that it can handle these HTTP
methods?  To add this functionality, we'll want to go back to our Controller
superclass and flush out the `getRecognizedHTTPMethods()` method.  Each subclass will inherit this method and its functionality.  The method will lean on certain methods from the Java reflection library being called in particular order:


* [`getClass()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#getClass--), which can be called on an object and returns a corresponding class object.
* [`getMethods()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethods--), which can be called on a class object and returns a list of method objects corresponding to the class's public methods.
* [`getReturnType()`](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#getReturnType--) and [`getName()`](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#getName--), both of which can be called on a method object and which return, respectively, a class object for a return type and a string for a method's name.

Let's see these reflection methods in action to bring functionality to the
`getRecognizedHTTPMethods()` of the Controller superclass.

```java
public abstract class Controller {

    //...omitted code...

    public Set<String> getRecognizedHTTPMethods() {
        Method[] classMethods = this.getClass().getMethods();
            /* Above, `this` refers to the inheriting subclass that is
               responding to `getRecognizedHTTPMethods()`. So if we call
               `new PhotosController().getRecognizedHTTPMethods(), at
               this point `classMethods` will include a `get()` method and
               a `post()` method.  We'll pass `classMethods` to a private
               method to do some sorting and manipulation... */
        return parseMethodsThatReturnResponseObjects(classMethods);
    }

    private Set<String> parseMethodsThatReturnResponseObjects(Method[] classMethods) {
        HashSet<String> parsedMethods = new HashSet<>();
            /* Below, we'll iterate through each of the methods in the
               `classMethods` variable and use `getReturnType()` to check
                if the method returns a Response object. For better or worse,
                here I've decided that returning a Response object is the
                key in the controller to signaling that this method corresponds
                to, and handles, an HTTP method.  In other words, we filter our
                methods by their return type as a convention to know which
                HTTP methods this Controller subclass responds to. */
        for (Method method : classMethods) {
            if (method.getReturnType() == Response.class) {
                  /* Once we've found a method that returns a Response object
                     (and so is responsible for handling an HTTP method), we'll
                     get the name of that method as a string, uppercase it, and
                     add it to our HashSet of parsedMethods. */
                parsedMethods.add(method.getName().toUpperCase());
            }
        }
            /* Going back to our PhotosController, if we now ask a
               PhotosController instance to respond to
               `getRecognizedHTTPMethods()`, it will return a tidy
               Set with `GET` and `POST` thanks to the reflection methods and
               sorting above. */
        return parsedMethods;
    }

}
```

Once a controller subclass implements the Controller superclass and inherits the `getRecognizedHTTPMethods()` method, it can now tell us to which HTTP methods it responds.  And we don't have to hardcode this list anywhere.  But how do we actually trigger one of these `get()` or `post()` methods if a client makes a valid request (e.g., the client made a `GET` request, and the relevant controller subclass has a `get()` method)?  Or, alternatively, how do we indicate a "[405 Method Not Allowed](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/405)" response is in order because the controller subclass does not know how to handle the HTTP method in the request (e.g., the client made a `POST` request to a valid path, but the corresponding controller subclass only has a `get()` method)?  

The means to pull this off lie in a new class, which we will call ControllerMapper.  To accomplish its role, an instance of ControllerMapper will rely on Java reflection methods, much like the Controller superclass did above.  To demonstrate how the ControllerMapper uses reflection, let's start out with our router. For this illustration, our router simply serves as a place where we map a path to a particular subclass controller, like so:

```java
public class Router {

    private Map<String, Class<Controller>> routeMap;

    public Router() {
        this.routeMap = new HashMap<>();
          /* Note that the values in our HashMap are class objects, brought
            to us by the `.class` property.  We are not actually instantating
            an instance of this class. */
        routeMap.put("/photos", PhotoController.class);
          /* Any other controller subclasses that we have defined can be
             added here, e.g.:
             routeMap.put("/comments", CommentsController.class) */
    }

      /* We'll expose a `getControllerFor()` method below to allow the
         ControllerMapper to grab the class object of a controller subclass
         when the time is right. */
    public Class<Controller> getControllerFor(String path) {
      return routeMap.get(path);
   }


}
```

With our router in place and mapping paths to a controller subclass object, let's imagine how our ControllerMapper will behave. I want it to have a public `handle()` method, to which we will pass as arguments a router instance, a Request object representing the client request, and a Response object representing the server response we are building to send back to the client. A ControllerMapper instance will "handle" it's responsibility by leaning on the Java reflection library to:

* (a) create an instance of the controller subclass requested (`getControllerForPathRequested()` below), and
* (b) ask the controller subclass to respond to a HTTP method (`askControllerToRespondToHTTPMethodRequested()` below). From there:
  * (1) If the controller subclass supports the HTTP method, get the method and call it (`callControllerMethod()`), or
  * (2) If the controller subclass does not support the method, then slap the Response object with a "401 Method Not Allowed" status code.

Here's a walk through of how I implemented the ControllerMapper.  The code has been simplified a bit for illustration purposes.  

```java
public class ControllerMapper extends Middleware {

    private Router router;
    private Request request;
    private Response response;
    private Controller controller;

    public void handle(Router router, Request request, Response response) {
        try {
            this.router = router
            this.request = request;
            this.response = response;
            getControllerForPathRequested();
            askControllerToRespondToHTTPMethodRequested();
        } catch (Exception e) {
            /* Java reflection methods can throw a lot of different
               Exceptions, so we definitely need a catch block somewhere. */
            e.printStackTrace();
        }
    }

    private void getControllerForPathRequested() throws NoSuchMethodException, InstantiationException, IllegalAccessException, java.lang.reflect.InvocationTargetException {
          /* Here we're asking our router to give us the class object
             corresponding to the path requested. */
        Class<Controller> controllerClass = router.getControllerFor(request.getPath());
          /* Once we've got the class object, we can use the reflection methods
             `getConstructor()` and `newInstance()` to generate an instance of
             the class without even knowing its name! We're relying on the
             router to know the name. */
        Constructor<Controller> controllerConstructor = controllerClass.getConstructor(Request.class, Response.class);
        controller = controllerConstructor.newInstance(request, response);
          /* On the line above, we've assigned our newly created controller
             subclass instance to our `controller` field so the methods below
             can access it. */
    }

    private void askControllerToRespondToHTTPMethodRequested() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
            /* Now we need to ask the controller subclass instance what HTTP
               methods it recognizes, using `getRecognizedHTTPMethods(),`
               which we coded up above. */
          (controller.getRecognizedHTTPMethods().contains(request.getHTTPMethod())) {
              /* If the controller subclass can handle the request, we call it
                 and mark the response with a "200 OK" status code. */
            callControllerMethod();
            response.statusCode = HTTPStatusCodes.OK;
        } else {
            /* If the controller subclass instance does not know how to handle
               the HTTP method requested, we'll add a "405 Method Not Allowed"
               status code to the response */
            response.statusCode = HTTPStatusCodes.MethodNotAllowed;
            response.addHeader(HTTPHeaders.Allow, controller.getRecognizedHTTPMethods());
        }
    }

    private void callControllerMethod() throws NoSuchMethodException, IllegalAccessException, java.lang.reflect.InvocationTargetException {
          /* If we've gotten this far, we know the controller subclass can
             handle the HTTP method requested.  We'll call the relevant
             method using the Java methods `getMethod()` and `invoke()`.
             Note in particular that `invoke()` is a bit picky and requires,
             as an argument, the object containing the method we are calling.
             That object, in our case, is the controller assigned to our
             controller field. */
        String controllerMethodToCall = request.getHTTPMethod().toLowerCase();
        Method methodToInvoke = controller.getClass().getMethod(controllerMethodToCall);
        methodToInvoke.invoke(controller);
    }

}
```

Phew.  That's a lot.  But with all of that in place, let's see if we've accomplished our goal of moving closer to a Rails-like router.  We'll do this through a comparison to a simple Rails example.  In Rails, if we wanted to have a PhotosController that responded to a `GET` and `POST` request, we might configure things along the following lines:

```ruby
#config/routes.rb
get 'photos', to: 'photos#index'
post 'photos', to: 'photos#create'

#photos_controller.rb
class PhotosController < ApplicationController

  def index
    # some stuff to respond to the request here
  end

  def show
    # some stuff to respond to the request here
  end

end
```

And now let's see what our configuration looks like working with the HTTP server implementation above.

```java
public class Router {

    private Map<String, Class<Controller>> routeMap;

    public Router() {
        this.routeMap = new HashMap<>();
        routeMap.put("/photos", PhotoController.class);
    }

}

public class PhotosController extends Controller {

    public PhotosController(Request request, Response response) {
        super(request, response);
    }

    public Response get() {
          /* Some stuff to respond to request here */
       return response;
    }

    public Response post() {
          /* Some stuff to respond to request here */
       return response;
    }

}
```

The Java configuration is slightly more verbose, but it's pretty close! While my homegrown HTTP server is humble and has limitations (like not being able to handle nested routes yet), I really like the simple configuration above as a way to map paths to controllers and their methods.  And it was fun to learn more about the Java reflection library and use it derive the simple-ish configuration above. I hope the examples above helped illustrate some of the reflection methods in action.  The complete code for the HTTP server can be found [here](https://github.com/breadoliveoilsalt/http-server-java), if you'd like to see more.   
