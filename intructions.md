**httpp Server in Go**

# Chapter 1. Servers

# 1.1 Server

We're building a fully-fledged web server from scratch on your local machine. The test suite will make HTTP requests to your local server over localhost. Your server will run in one terminal, while you submit tests with the Boot.dev CLI in another terminal.

Setup
Tools You'll Need
A code editor. I use VS code, but you can use whatever you're comfortable with.
A command line. I work on macOS/Linux, so my instructions will be in Bash. I recommend WSL 2 if you're on Windows so you can still use Linux commands.
The Go toolchain with version 1.22+.
The Boot.dev CLI to run the tests. Go ahead and install it following the instructions in the README, then run bootdev login to authenticate.
The lessons in this course require at least version 1.22 of Go. If you're using an older version, you'll run into some frustrating issues!

Set up Your Project
Create a new GitHub/GitLab repository for your Chirpy project, and clone it down onto your local machine. Use go mod init to create a new Go module for the project, and add a main.go file. That's where you'll be writing your code for each assignment.

Do not delete your work after each assignment! Each lesson will build upon the previous ones so we'll be reusing a lot of code.

Assignment
The Go standard library makes it easy to build a simple server. Your task is to build and run a server that binds to localhost:8080 and always responds with a 404 Not Found response.

Steps
Create a new http.ServeMux
Create a new http.Server struct.
Use the new "ServeMux" as the server's handler
Set the .Addr field to ":8080"
Use the server's ListenAndServe method to start the server
Build and run your server (e.g. go build -o out && ./out)
Open <http://localhost:8080> in your browser. You should see a 404 error because we haven't connected any handler logic yet. Don't worry, that's what is expected for the tests to pass for now.
Being a programmer means reading documentation and searching for examples. This is an essential skill because a professional in any field should never stop learning. In fact, the best way to learn is on the clock. Check out the docs provided for examples and be sure to read the http package's Overview as a primer.
Run and submit the CLI tests using the Boot.dev CLI tool in another terminal window while your server is still running.

# 1.2 Fileservers

A fileserver is a kind of simple web server that serves static files from the host machine. Fileservers are often used to serve static assets for a website, things like:

HTML
CSS
JavaScript
Images
Assignment
The Go standard library makes it super easy to build a simple fileserver. Build and run a fileserver that serves a file called index.html from its root at <http://localhost:8080>. That file should contain this HTML:

<html>
  <body>
    <h1>Welcome to Chirpy</h1>
  </body>
</html>

Steps
Add the HTML code above to a file called index.html in the same root directory as your server
Use the http.NewServeMux's .Handle() method to add a handler for the root path (/).
Use a standard http.FileServer as the handler
Use http.Dir to convert a filepath (in our case a dot: . which indicates the current directory) to a directory for the http.FileServer.
Re-build and run your server
Test your server by visiting <http://localhost:8080> in your browser

# 1.3 Serving Images

You may be wondering how the fileserver knew to serve the index.html file to the root of the server. It's such a common convention on the web to use a file called index.html to serve the webpage for a given path, that the Go standard library's FileServer does it automatically.

When using a standard fileserver, the path to a file on disk is the same as its URL path. An exception is that index.html is served from / instead of /index.html.

Try It Out
Run your chirpy server again, and open <http://localhost:8080/index.html> in a new browser tab. You'll notice that you're redirected to <http://localhost:8080/>.

This works for all directories, not just the root!

For example:

/index.html will be served from /
/pages/index.html will be served from /pages
/pages/about/index.html will be served from /pages/about
Alternatively, try opening a URL that doesn't exist, like <http://localhost:8080/doesntexist.html>. You'll see that the fileserver returns a 404 error.

Assignment
Let's serve another type of file from our server: an image. Chirpy has a slick logo, and we need to serve it so that our users can load it in their browsers and mobile apps.

Download the Chirpy logo from below and add it to your project directory.

Configure its filepath so that it's accessible from this URL:

<http://localhost:8080/assets/logo.png>


# 1.4 Workflow Tips

Servers are interesting because they're always running. A lot of the code we've written in Boot.dev up to this point has acted more like a command line tool: it runs, does its thing, and then exits.

Servers are different. They run forever, waiting for requests to come in, processing them, sending responses, and then waiting for the next request. If they didn't work this way, websites and apps would be down and unavailable all the time!

Debugging a Server
Debugging a CLI app is simple:

Write some code.
Build and run the code.
See if it did what you expected.
If it didn't, add some logging or fix the code, and go back to step 2.
Debugging a server is a little different. The simplest way (minimal tooling) is to:

Write some code.
Build and run the code.
Send a request to the server using a browser or some other HTTP client.
See if it did what you expected.
If it didn't, add some logging or fix the code, and go back to step 2.
Make sure you're testing your server by hitting endpoints in the browser before submitting your answers.

Restarting a Server
I usually use a single command to build and run my servers, assuming I'm in my main package directory:

go run .

This builds the server and runs it in one command.

To stop the server, I use ctrl+c. This sends a signal to the server, telling it to stop. The server then exits.

To start it again, I just run the same command.

Alternatively, you can compile a binary and run it instead

go build -o out && ./out

CLI Tip
If you didn't know, you can continuously press the up arrow key on the command line to see the commands you've previously run. That way you don't need to re-type commands that you use often!


# 1.5 Custom Handlers

In the previous exercise, we used the http.FileServer function, which simply returns a built-in http.Handler.

An http.Handler is just an interface:

type Handler interface {
 ServeHTTP(ResponseWriter, *Request)
}

Any type with a ServeHTTP method that matches the http.HandlerFunc signature above is an http.Handler. Take a second to think about it: it makes a lot of sense! To handle an incoming HTTP request, all a function needs is a way to write a response and the request itself.

Assignment
Let's add a readiness endpoint to the Chirpy server! Readiness endpoints are commonly used by external systems to check if our server is ready to receive traffic.

The endpoint should be accessible at the /healthz path using any HTTP method.

The endpoint should simply return a 200 OK status code indicating that it has started up successfully and is listening for traffic. The endpoint should return a Content-Type: text/plain; charset=utf-8 header, and the body will contain a message that simply says "OK" (the text associated with the 200 status code).

Later this endpoint can be enhanced to return a 503 Service Unavailable status code if the server is not ready.

1. Add the Readiness Endpoint
I recommend using the mux.HandleFunc to register your handler. Your handler can just be a function that matches the signature of http.HandlerFunc:

func(http.ResponseWriter, *http.Request)

Your handler should do the following:

Write the Content-Type header
Write the status code using w.WriteHeader
Write the body text using w.Write
2. Update the Fileserver Path
Now that we've added a new handler, we don't want potential conflicts with the fileserver handler. Update the fileserver to use the /app/ path instead of /.

Not only will you need to mux.Handle the /app/ path, you'll also need to strip the /app prefix from the request path before passing it to the fileserver handler. You can do this using the http.StripPrefix function.

# 1.6 Handler Review

Handler
An http.Handler is any defined type that implements the set of methods defined by the Handler interface, specifically the ServeHTTP method.

type Handler interface {
 ServeHTTP(ResponseWriter, *Request)
}

The ServeMux you used in the previous exercise is an http.Handler.

You will typically use a Handler for more complex use cases, such as when you want to implement a custom router, middleware, or other custom logic.

HandlerFunc
type HandlerFunc func(ResponseWriter, *Request)

You'll typically use a HandlerFunc when you want to implement a simple handler. The HandlerFunc type is just a function that matches the ServeHTTP signature above.

Why This Signature?
The Request argument is fairly obvious: it contains all the information about the incoming request, such as the HTTP method, path, headers, and body.

The ResponseWriter is less intuitive in my opinion. The response is an argument, not a return type. Instead of returning a value all at once from the handler function, we write the response to the ResponseWriter.

# Chapter 2. Routing

# 2.1 Middleware

Middleware is a way to wrap a handler with additional functionality. It is a common pattern in web applications that allows us to write DRY code.

For example, we can write a middleware that logs every request to the server. We can then wrap our handler with this middleware and every request will be logged without us having to write the logging code in every handler.

To do that, we can write the middleware function like this:

func middlewareLog(next http.Handler) http.Handler {
 return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
  log.Printf("%s %s", r.Method, r.URL.Path)
  next.ServeHTTP(w, r)
 })
}

Then, any handler that needs logging can be wrapped by this middleware function:

mux.Handle("/app/", middlewareLog(handler))


# 2.2 Stateful Handlers

It's frequently useful to have a way to store and access state in our handlers. For example, we might want to keep track of the number of requests we've received, or we may want to pass around an open connection to a database, or credentials to an API.

Assignment
The product managers at Chirpy want to know how many requests are being made to serve our homepage - in essence, they want to know how many people are viewing the site!

They have asked for a simple HTTP endpoint they can hit to get the number of requests that have been processed. It will return the count as plain text in the response body.

For now, they just want the number of requests that have been processed since the last time the server was started, we don't need to worry about saving the data between restarts.

Steps
Create a struct in main.go that will hold any stateful, in-memory data we'll need to keep track of. In our case, we just need to keep track of the number of requests we've received.
type apiConfig struct {
 fileserverHits atomic.Int32
}

The atomic.Int32 type is a really cool standard-library type that allows us to safely increment and read an integer value across multiple goroutines (HTTP requests).

Next, write a new middleware method on a *apiConfig that increments the fileserverHits counter every time it's called. Here's the method signature I used:
func (cfg*apiConfig) middlewareMetricsInc(next http.Handler) http.Handler {
 // ...
}

The atomic.Int32 type has an .Add() method, use it to safely increment the number of fileserverHits.

Wrap the http.FileServer handler with the middleware method we just wrote. For example:
mux.Handle("/app/", apiCfg.middlewareMetricsInc(handler))

Create a new handler that writes the number of requests that have been counted as plain text in this format to the HTTP response:
Hits: x

Where x is the number of requests that have been processed. This handler should be a method on the *apiConfig struct so that it can access the fileserverHits data.

Register that handler with the serve mux on the /metrics path.
Finally, create and register a handler on the /reset path that, when hit, will reset your fileserverHits back to 0.
It should follow the same design as the previous handlers.

Remember, similar to the metrics endpoint, /reset will need to be a method on the *apiConfig struct so that it can also access the fileserverHits

# 2.3 Routing

The Go standard library has a lot of powerful HTTP features and, as of version 1.22, comes equipped with method-based pattern matching for routing.

In this lesson, we are going to limit which endpoints are available via which HTTP methods. In our current implementation, we can use any HTTP method to access any endpoint. This is not ideal.

There are powerful routing libraries like Gorilla Mux and Chi, however, the course will assume you are using Go's standard library. Just know that it isn't your only option!
Try It!
Run this command to send an empty POST request to your running server:

curl -X POST <http://localhost:8080/healthz>

You should get an OK response - but we want this endpoint to only be available via GET requests!

Method Specific Routing
Using the Go standard library, you can specify a method like this: [METHOD ][HOST]/[PATH]. For example:

mux.HandleFunc("POST /articles", handlerArticlesCreate)
mux.HandleFunc("DELETE /articles", handlerArticlesDelete)

Assignment
Update the following paths to only accept GET requests:
/healthz
/metrics
When a request is made to one of these endpoints with a method other than GET, the server should return a 405 (Method Not Allowed) response (this is handled automatically!).

Update the /reset endpoint to only accept POST requests.

# 2.4 Patterns

A pattern is a string that specifies the set of URL paths that should be matched to handle HTTP requests. Go's ServeMux router uses these patterns to dispatch requests to the appropriate handler functions based on the URL path of the request. As we saw in the previous lesson, patterns help organize the handling of different routes efficiently.

As previously mentioned, patterns generally look like this: [METHOD ][HOST]/[PATH]. Note that all three parts are optional.

Rules and Definitions
Fixed URL Paths
A pattern that exactly matches the URL path. For example, if you have a pattern /about, it will match the URL path /about and no other paths.

Subtree Paths
If a pattern ends with a slash /, it matches all URL paths that have the same prefix. For example, a pattern /images/ matches /images/, /images/logo.png, and /images/css/style.css. As we saw with our /app/ path, this is useful for serving a directory of static files or for structuring your application into sub-sections.

Longest Match Wins
If more than one pattern matches a request path, the longest match is chosen. This allows more specific handlers to override more general ones. For example, if you have patterns / (root) and /images/, and the request path is /images/logo.png, the /images/ handler will be used because it's the longest match.

Host-Specific Patterns
We won't be using this but be aware that patterns can also start with a hostname (e.g., <www.example.com/>). This allows you to serve different content based on the Host header of the request. If both host-specific and non-host-specific patterns match, the host-specific pattern takes precedence.

If you're interested, you can read more in the ServeMux docs.

# Chapter 3. Architecture

# 3.1 Monoliths and Decoupling

"Architecture" in software can mean many different things, but in this lesson, we're talking about the high-level architecture of a web application from a structural standpoint. More specifically, we are concerned with the separation (or lack thereof) between the back-end and the front-end.

When we talk about "coupling" in this context, we're talking about the coupling between the data and the presentation logic of that data. Loosely speaking, when I say "a tightly coupled front-end and back-end", what I mean is:

Front-End: the Presentation Logic
If it's a web app, then this is the HTML, CSS, and JavaScript that is served to the browser which will then be used to render any dynamic data. If it's a mobile app, then this is the compiled code that is downloaded on the mobile device.

Back-End: Raw Data
For an app like YouTube, this would be videos and comments. For an app like Twitter, this might be tweets and users data. You can't embed the YouTube videos directly into the Youtube app, because a user's feed changes each time they open the app. The app downloads new raw data from Google's back-end each time the app is opened.

Monolithic

A monolith is a single, large program that contains all of the functionality for both the front-end and the back-end of an application. It's a common architecture for web applications, and it's what we're building here in this course.

Sometimes monoliths host a REST API for raw data (like JSON data) within a subpath, like /api as shown in the image. That said, there are even more tightly coupled kinds of monoliths that inject the dynamic data directly into the HTML as well. The nice thing about separate data endpoints is that they can be consumed by any client, (like a mobile app) and not just the website. That said, injection is typically more performant, so it's a trade-off. WordPress and other website builders typically work this way.

Decoupled

A "decoupled" architecture is one where the front-end and back-end are separated into different codebases. For example, the front-end might be hosted by a static file server on one domain, and the back-end might be hosted on a subdomain by a different server.

Depending on whether or not a load balancer is sitting in front of a decoupled architecture, the API server might be hosted on a separate domain (as shown in the image) or on a subpath, as shown in the monolithic architecture. A decoupled architecture allows for either approach.

Assignment
For now, Chirpy is technically a monolith. That said we are keeping all the API logic decoupled in the sense that it will be served from its own namespace (path prefix). We serve the website from the app path, and we'll be serving the API from the /api path.

Let's move our non-website endpoints to the /api namespace in our routing.

To do this, prepend /api to the beginning of each of our API (non fileserver) endpoints.

# 3.2 Which Is Better?

There is always a trade-off.

Pros for Monoliths
Simpler to get started with
Easier to deploy new versions because everything is always in sync
In the case of the data being embedded in the HTML, the performance can result in better UX and SEO
Pros for Decoupled Architectures
Easier, and probably cheaper, to scale as traffic grows
Easier to practice good separation of concerns as the codebase grows
Can be hosted on separate servers and using separate technologies
Embedding data in the HTML is still possible with pre-rendering (similar to how Next.js works), it's just more complicated
Can We Have the Best of Both Worlds?
Perhaps. My recommendation to someone building a new application from scratch would be to start with a monolith, but to keep the API and the front-end decoupled logically within the project from the start (like we're doing with Chirpy).

That way, our app is easy to get started with, but we can migrate to a fully decoupled architecture later if we need to.

# 3.3 Admin Namespace

Let's add an "admin" namespace. This is where we'll put endpoints intended for internal administrative use. Note: there's nothing inherently more secure about this namespace, it's just an organizational structure.

Assignment
Swap out the GET /api/metrics endpoint, which just returns plain text, for a GET /admin/metrics that returns HTML to be rendered in the browser. Use this template with fmt.Sprintf():
<html>
  <body>
    <h1>Welcome, Chirpy Admin</h1>
    <p>Chirpy has been visited %d times!</p>
  </body>
</html>

Where %d is replaced with the number of times the page has been loaded.

Make sure you use the Content-Type header to set the response type to text/html so that the browser knows how to render it.
Try loading <http://localhost:8080/admin/metrics> in your browser, and in another tab load <http://localhost:8080/app> a few times. Refreshing the admin page should show the updated count.
Update the POST /api/reset to POST /admin/reset. Its functionality should not change.

# 3.4 Deployment Options

We won't go in-depth with deployment instructions right now; that said, let's talk about how our choice of project architecture affects our deployment options, and how we could deploy our application in the future. We'll only talk about cloud deployment options here, and by the "cloud" I'm just referring to a remote server that's managed by a third-party company like Google or Amazon.

xkcd
Using a cloud service to deploy applications is super common these days because it's easy, fast, and cheap.

That said, it's still possible to deploy to a local or on-premise server, and some companies still do that, but it's not as common as it used to be.

Monolithic Deployment
Deploying a monolith is straightforward. Because your server is just one program, you just need to get it running on a server that's exposed to the internet and point your DNS records to it.

You could upload and run it on classic server, something like:

AWS EC2
GCP Compute Engine (GCE)
Digital Ocean Droplets
Azure Virtual Machines
Alternatively, you could use a platform that's specifically designed to run web applications, like:

Heroku
Google App Engine
Fly.io
AWS Elastic Beanstalk
Decoupled Deployment
With a decoupled architecture, you have two different programs that need to be deployed. You would typically deploy your back-end to the same kinds of places you would deploy a monolith.

For your front-end server, you can do the same, or you can use a platform that's specifically designed to host static files and server-side rendered front-end apps, something like:

Vercel
Netlify
GitHub Pages
Because the front-end bundle is likely just static files, you can host it easily on a CDN (Content Delivery Network) inexpensively.

More Powerful Options
If you want to be able to scale your application up and down in specific ways, or you want to add other back-end servers to your stack, you might want to look into container orchestration options like Kubernetes and Docker Swarm.

Don't Worry About All This Stuff!
I'm trying to gently introduce you to some popular technologies and how they work together, but you don't need to memorize all of these products and options.

# 4. HTTP Clients

# 4,1 HTTP Clients

So far, you have probably been using a browser to test your server. That works fine with simple GET requests (the kind of request a browser sends when you type a URL into the address bar), but it's not very useful for any other HTTP methods or requests with custom headers and bodies.

Debugging Your Endpoints
Servers are built to be used by clients. As you develop your code, you should be using a tool that makes sending one-off requests to your server easy! Here are some of my favorites:

REST Client for VS Code
Postman for VS Code
cURL
Postman
Use whichever client you like, but make sure you're using one!

# 4.2 JSON

Hopefully, by now you already know what JSON is. If not, you should go back and take the Learn HTTP Clients course here first.

What you may be new to is handling and parsing JSON on the server side, rather than sending it as a client.

If you want to take a super deep dive into JSON in Go, then you can read this post here. With that in mind, you don't need to! I'll give you the relevant info below.

Decode JSON Request Body
It's very common for POST requests to send JSON data in the request body. Here's how you can handle that incoming data:

{
  "name": "John",
  "age": 30
}

func handler(w http.ResponseWriter, r *http.Request){
    type parameters struct {
        Name string `json:"name"`
        Age int `json:"age"`
    }

    decoder := json.NewDecoder(r.Body)
    params := parameters{}
    err := decoder.Decode(&params)
    if err != nil {
  log.Printf("Error decoding parameters: %s", err)
  w.WriteHeader(500)
  return
    }
    // params is a struct with data populated successfully
    // ...
}

The struct tags (e.g., `json:"name"`) indicate how the keys in the JSON should be mapped to the struct fields. The struct fields themselves must be exported (start with a capital letter) if you want them parsed.

decoder.Decode() will return an error if the JSON is invalid or has the wrong types, and any missing fields will simply have their values in the struct set to their zero value.

Encode JSON Response Body
func handler(w http.ResponseWriter, r *http.Request){
    // ...

    type returnVals struct {
        CreatedAt time.Time `json:"created_at"`
        ID int `json:"id"`
    }
    respBody := returnVals{
        CreatedAt: time.Now(),
        ID: 123,
    }
    dat, err := json.Marshal(respBody)
 if err != nil {
   log.Printf("Error marshalling JSON: %s", err)
   w.WriteHeader(500)
   return
 }
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(200)
    w.Write(dat)
}

Again, we use struct tags to specify how the field names will be encoded in the JSON data. If you omit the tags, the keys will be encoded as the same names of struct fields (e.g., CreatedAt, ID).

Assignment
At Chirpy, we have a silly rule that says all Chirps must be 140 characters long or less.

Add a new endpoint to the Chirpy API that accepts a POST request at /api/validate_chirp. It should expect a JSON body of this shape:

{
  "body": "This is an opinion I need to share with the world"
}

If any errors occur, it should respond with an appropriate HTTP status code and a JSON body of this shape:

{
  "error": "Something went wrong"
}

For example, if the Chirp is too long, respond with a 400 code and this body:

{
  "error": "Chirp is too long"
}

If the Chirp is valid, respond with a 200 code and this body:

{
  "valid": true
}

The JSON here has been prettified. The actual response body won't have newlines or spaces between the key and value. They'll look more like {"valid":true}
