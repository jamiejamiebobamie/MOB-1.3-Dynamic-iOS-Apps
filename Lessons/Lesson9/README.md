<!-- Run this slideshow via the following command: -->
<!-- reveal-md README.md -w -->


<!-- .slide: class="header" -->

# Building a Network Layer

## [Slides](https://make-school-courses.github.io/MOB-1.3-Dynamic-iOS-Apps/Slides/Lesson9/README.html ':ignore')

<!-- > -->

## Why you should know this

As a professional iOS developer, you will want to...

1. Design and implement code that:
- Is organized and easy to understand
- Is easy for you and your team mates to debug, extend and maintain

2. Demonstrate to potential employers your understanding of best practices for organizing code, as well as core programming principles and design patterns.

<!-- > -->

## Learning Objectives

At the end of this class, you should be able to...

1. Design and construct a network layer that optimizes:
  - modularity and reusability
  - maintainability
  - extendability
2. Demonstrate understanding of key application design principles:
  - Separation of Concerns (SoC)
  - Code reuse

<!-- > -->

## Design Principles

As a design pattern, MVC seeks to promote two important design principles fundamental to OOP:

1. **Separation of Concerns (SoC)** - modular approach to constructing an application in which code can be separated into logical sections, each addressing separate areas of functional behavior (concerns).

 - MVC can separate content from presentation, and data-processing (model) from content.
 - Service-oriented design can separate concerns into services.

<!-- > -->

 2. **Reusability** - If correctly implemented, view and model layers can easily be composed of reusable, modular components (though controllers are seldom reusable).

<!-- > -->

### Project Organization

Structure and organization are key contributors to effective modularized architecture. Creating groups folder is a great place to start.

*Controllers, views, and model are only suggestions - feel free to name yours whatever makes sense in your project.*

![syntax](assets/project_folders.png)

<!-- > -->

### Service Objects

The core component of an API Layer is the **Service** or **API object.**

This object's job is to:
- **Fetch, post and process** data to and from the target web services
- **Serialize JSON** data for manipulation and presentation
- Provide constructs for **handling the successful or failed** state of web service requests and responses

<!-- > -->

## The Request Builder

The **Builder** design pattern is a type of **Creational** design pattern that is used to create complex objects step-by-step.

It offers:
- **Flexibility** - Can easily create different representations of the same complex object
- __Simplicity__ - It simplifies the creation of a complex object

<!-- > -->

HTTP request methods GET, POST, DELETE, and so on, are constructed using parameters common to them all.

HTTP requests offer a prime opportunity to employ the Builder pattern to create different types of request objects with commonly shared parameters.

<!-- > -->

Instead of rewriting the parameters for a separate request object for each HTTP method, a more efficient design is to design a single **RequestBuilder** object that reuses the commonly shared request parameters, then call a separate function on the RequestBuilder object to create a request for a GET or a POST, respectively.

<!-- > -->

## Moviefy App

Two approaches:

[Moviefy base implementation](https://github.com/Make-School-Courses/MOB-1.3-Dynamic-iOS-Apps/blob/master/Lessons/Lesson9/assignments/moviefy-base.md)

[Moviefy generic implementation](https://github.com/Make-School-Courses/MOB-1.3-Dynamic-iOS-Apps/blob/master/Lessons/Lesson9/assignments/moviefy-plus.md)

Suggestion: Do both in order to identify how they differ.

<!--
## In Class Activity I (25 min)

Required resources:
1. Download the base app, [PhotoMatic](https://github.com/VanderDev1/PhotoMatic_L09.git)

### - Individual Activity -

__Scenario:__
- Congratulations! You just got hired at a new firm. But you "inherited" code created by several previous developers who no longer work at the company.
- The app was written in an older version of Swift.
- It was originally created as a quick “prototype” (i.e., it was not designed with maintenance and extension in mind), and it was developed by engineers new to both iOS and Swift.

**TODO:** Using what you’ve learned so far about iOS networking, your assignment is to:
- Refactor the code so that it (a) is scalable, and (b) adheres to the tenets of MVC
- Where you see quick and practical opportunities, update the code to Swift 4 constructs, including implementing model objects with the `Codable` interface, properly handling Optionals and errors, and so on...

**Steps to Complete**

1. Validate the App - Insert your own Flickr API Key and confirm the *current working state* of the app.

2. Refactor (or move) all **network calls** and **JSON processing functions** into the **PhotoFetchService**.

**TIP:** Simply search for the `//TODO:` comments to see key refactoring points

3. Recreate the model object (`Photo.swift` class) to implement the Codable interface.
- Note the impact this change may have to other part of the code base.

4. (If time permits) Review any other opportunities to improve the app (i.e., handling optionals, etc.). If time permits, make those improvements.
-->



<!--
## HTTP Post Requests (15 mins)

To add a new item to a web service, we use the HTTP protocol's **POST method.**

Implementing a POST request is a bit like performing a GET request in reverse, except that for a POST you will need to supply additional parameters to the URLRequest object.

Commonly required parameters include:

- The __httpMethod type__ (i.e., “POST”)
- The __content type__ (JSON, in our case)
- Or any other __headers__ specifically required by your target web service API (a valid API Key, for example)


### Step 1: Set Up the Session and Requests

Just as we did with our HTTP GET request, we first need to create and configure a **URLSession** and a **URLRequest** object that points to our target web service **URL.**

```Swift
  let session = URLSession.shared
  let url = URL(string: "https://<your_web_service_url>")
  var request = URLRequest(url: url!)
```

**Note:** This example uses the alternate ___shared___ URLSession type: `URLSession.shared`


### Step 2: Configure the Request

#### Specify the httpMethod type

For any web service request other than GET (i.e., POST, PUT, PATCH, DELETE), we need to specify the **httpMethod** to invoke.

Since we are performing a POST here, we will set the httpMethod property to `urlRequest = "POST"`.

```Swift
  request.httpMethod = "POST"
```

#### Specify Headers

Next, use the URLRequest `setValue(_:forHTTPHeaderField:)` method to set the values of any HTTP headers you want to provide (except the `Content-Length` header. The session automatically figures out content length  from the size of your data).

___`Content-Type` Header Field___

We use `Content-Type` header to indicate to the web service API the type of data we are sending.

In our case, we want to set the `Content-Type` to `JSON`.

```Swift
  request.setValue(“application/json”, forHTTPHeaderField: “Content-Type”)
```

___`Accept` Header Field___

The `Accept` request header field is used to specify certain **media types** that are acceptable for the **response** object returned by the web service.

We want our response to be returned as JSON, so we set the `Accept` request header field to return JSON.

```Swift
  request.setValue(“application/json”, forHTTPHeaderField: “Accept”)
```

___Other Header Fields___

Follow the same process of using the URLRequest `setValue(_:forHTTPHeaderField:)` method to supply all header fields required for communicating with your target web service.

A valid API Key is commonly required for the "Authorization" header field:

```Swift
    request.setValue("<insert_valid_API-KEY_here>", forHTTPHeaderField: “Authorization”)
```


### Step 3: Convert Data to JSON Format

To convert our data object to the JSON format, we will use a built-in function of `JSONSerialization:`

 ```Swift
 JSONSerialization.data(withJSONObject obj: Any, options opt: JSONSerialization.WritingOptions = []) throws
```

*What exactly does this function do?* Here is a brief excerpt from Apple's description:

*Generate JSON data from a Foundation object. If the object will not produce valid JSON then an exception will be thrown. Setting the NSJSONWritingPrettyPrinted option will generate JSON with whitespace designed to make the output more readable. If that option is not set, the most compact possible JSON will be generated. If an error occurs, the error parameter will be set and the return value will be nil...*

We convert our data into JSON, then include the converted JSON data into the httpBody of the URL request and handle any errors thrown.

```Swift
let parameters: [String: Any] = [“foo”: “bar”, “numbers”: [1, 2, 3, 4, 5]]
       do {
           let jsonParams = try JSONSerialization.data(withJSONObject: parameters, options: [])
           postRequest.httpBody = jsonParams
       } catch { print(“Error: unable to add parameters to POST request.“)}
```

The `options` array is left blank here, but it can be used to print or to sort the output (see Apple references below for more details).


### Step 4:  Execute Request

Finally, the dataTask will execute our POST request with the our specified header values, and its completion block closure will be executed after the response is returned from the web service.

```Swift
session.dataTask(with: request, completionHandler: { (data, response, error) -> Void in
           if error != nil { print(“POST Request: Communication error: \(error!)“) }
           if data != nil {
               do {
                   let resultObject = try JSONSerialization.jsonObject(with: data!, options: [])
                   DispatchQueue.main.async(execute: {
                       print(“Results from POST request:\n\(resultObject)“)
                   })
               } catch {
                   DispatchQueue.main.async(execute: {
                       print(“Unable to parse JSON response”)
                   })
               }
           } else {
               DispatchQueue.main.async(execute: {
                   print(“Received empty response.“)
               })
           }
       }).resume()
```

### Put It Altogether

The complete code for an HTTP POST request function would resemble this:

```Swift
        let session = URLSession.shared
        let url = URL(string: "https://<your_web_service_url>")
        var request = URLRequest(url: url!)

        request.httpMethod = “POST”
        request.setValue(“application/json”, forHTTPHeaderField: “Content-Type”)
        request.setValue(“application/json”, forHTTPHeaderField: “Accept”)
        request.setValue(“<insert_valid_API-KEY_here>”, forHTTPHeaderField: “Authorization”)

        let parameters: [String: Any] = [“foo”: “bar”, “numbers”: [1, 2, 3, 4, 5]]
            do {
                let jsonParams = try JSONSerialization.data(withJSONObject: parameters, options: [])
                request.httpBody = jsonParams
            } catch { print(“Error: unable to add parameters to POST request.“)}

        session.dataTask(with: request, completionHandler: { (data, response, error) -> Void in
            if error != nil { print(“POST Request: Communication error: \(error!)“) }

            if data != nil {
                do {
                    let resultObject = try JSONSerialization.jsonObject(with: data!, options: [])
                    DispatchQueue.main.async(execute: {
                        print(“Results from POST request:\n\(resultObject)“)
                    })
                } catch {
                    DispatchQueue.main.async(execute: {
                      print(“Unable to parse JSON response”)
                    })
                }
            } else {
                DispatchQueue.main.async(execute: {
                    print(“Received empty response.“)
                })
        }
        }).resume()
```
-->


<!--
## Challenge

### Required Resources:
1. The pre-made [starter app for Lesson 10](https://github.com/VanderDev1/Lesson10.git)

2. [JSONPlaceholder API](https://jsonplaceholder.typicode.com)  - a free "Fake Online REST API for Testing and Prototyping"

For this challenge, we will ___simulate___ an HTTP **POST** request to the JSONPlaceholder API's `todos` endpoint.

**TIP:** The JSONPlaceholder API's home page provides a link to their "How to" page, which provides clues about how to create a successful POST request to their `todos` endpoint. But the code featured there is *JavaScript, not Swift...*

### Your Assignment:

Inside the `URLSessionApiService` class of the pre-made [starter app for Lesson 10](https://github.com/VanderDev1/Lesson10.git), you are to **create a function** that **executes a POST request** to the https://jsonplaceholder.typicode.com/todos endpoint.



___Hint:___ The app is already set up with a button to invoke your POST request function

**TODO #1 -** Create the POST request function:

Your POST request should pass data for these parameters:
- “userId"
- “title”
-  “completed”

Appending an actual user id to the endpoint shows you the data structure and parameters needed...

For example, running the following URL for `"userId" = 6` in a browser...

https://jsonplaceholder.typicode.com/todos/6

...will shows the fields and current status for userId 6:

```Swift
{
 “userId”: 6,
 “id”: 101,
 “title”: “explicabo enim cumque porro aperiam occaecati minima”,
 “completed”: false
}
```

**TODO #2 -** Validate Results:

- errors or successful results<sup>[2](#footnote2)</sup> can be found in your Xcode Debug log, so be sure to print messages to the log to signify *success* or *failure* conditions...

<!--
3. In constructing your project, follow practices we learned for constructing an API Layer...

- Create a Request Builder class to supply the configured request for the POST request (for now -- we will expand this class later in the course)

5. Add a Unit Test for asserting 1 failed/error condition

-->

<!-- > -->

## Additional Resources

- [Separation of Concerns (withJSONObject) - from Wikepedia](https://en.wikipedia.org/wiki/Separation_of_concerns)
- Apple on JSONSerialization Reading and Writing Options:</br>
- [Writing Options - from Apple](https://developer.apple.com/documentation/foundation/jsonserialization/writingoptions)
- [Reading Options - from Apple](https://developer.apple.com/documentation/foundation/jsonserialization/readingoptions)
- [Writing a network layer - Protocol Oriented](https://medium.com/flawless-app-stories/writing-network-layer-in-swift-protocol-oriented-approach-4fa40ef1f908)
- [Writing network layer](https://medium.com/@rinradaswift/networking-layer-in-swift-5-111b02db1639)
