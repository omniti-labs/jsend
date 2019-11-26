# JSend

* **What?** - Put simply, JSend is a specification that lays down some rules for how [JSON](http://json.org/) responses from web servers should be formatted. JSend focuses on application-level (as opposed to protocol- or transport-level) messaging which makes it ideal for use in [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer)-style applications and APIs.
* **Why?** - There are lots of web services out there providing JSON data, and each has its own way of formatting responses. Also, developers writing for JavaScript? front-ends continually re-invent the wheel on communicating data from their servers. While there are many common patterns for structuring this data, there is no consistency in things like naming or types of responses. Also, this helps promote happiness and unity between backend developers and frontend designers, as everyone can come to expect a common approach to interacting with one another.
* **Hold on now, aren't there already specs for this kind of thing?** - Well... no. While there are a few handy specifications for dealing with JSON data, most notably [Douglas Crockford](http://www.crockford.com/)'s [JSONRequest](http://www.json.org/JSONRequest.html) proposal, there's nothing to address the problems of general application-level messaging. More on this later.
* **(Why) Should I care?** - If you're a library or framework developer, this gives you a consistent format which your users are more likely to already be familiar with, which means they'll already know how to consume and interact with your code. If you're a web app developer, you won't have to think about how to structure the JSON data in your application, and you'll have existing reference implementations to get you up and running quickly.

## So how does it work? 
A basic JSend-compliant response is as simple as this:
```
{
    status : "success",
    data : {
        "post" : { "id" : 1, "title" : "A blog post", "body" : "Some useful content" }
     }
}
```
When setting up a JSON API, you'll have all kinds of different types of calls and responses. JSend separates responses into some basic types, and defines required and optional keys for each type:

<table>
<tr><th>Type</td><th>Description</th><th>Required Keys</th><th>Optional Keys</td></tr>
<tr><td>success</td><td>All went well, and (usually) some data was returned.</td><td>status, data</td><td></td></tr>
<tr><td>failure</td><td>There was a problem with the data submitted, or some pre-condition of the API call wasn't satisfied</td><td>status, data</td><td></td></tr>
<tr><td>error</td><td>An error occurred in processing the request, i.e. an exception was thrown</td><td>status, message</td><td>code, data</td></tr>
</table>

## Example response types 

### Success ### 
When an API call is successful, the JSend object is used as a simple envelope for the results, using the data key, as in the following:
#### GET /posts.json: ####
```
{
    status : "success",
    data : {
        "posts" : [
            { "id" : 1, "title" : "A blog post", "body" : "Some useful content" },
            { "id" : 2, "title" : "Another blog post", "body" : "More content" },
        ]
     }
}
```
#### GET /posts/2.json: ####
```
{
    status : "success",
    data : { "post" : { "id" : 2, "title" : "Another blog post", "body" : "More content" }}
}
```
#### DELETE /posts/2.json: ####
```
{
    status : "success",
    data : null
}
```
Required keys:

* status: Should always be set to "success".
* data: Acts as the wrapper for any data returned by the API call. If the call returns no data (as in the last example), data should be set to null.

### failure ### 
When an API call is rejected due to invalid data or call conditions, the JSend object's data key contains an object explaining what went wrong, typically a hash of validation errors. For example:
#### POST /posts.json (with data body: "Trying to creating a blog post"): ####
```
{
    "status" : "failure",
    "data" : { "title" : "A title is required" }
}
```
Required keys:

* status: Should always be set to "failure".
* data: Again, provides the wrapper for the details of why the request failed. If the reasons for failure correspond to POST values, the response object's keys SHOULD correspond to those POST values.

### Error ### 
When an API call fails due to an error on the server. For example:
#### GET /posts.json: ####
```
{
    "status" : "error",
    "message" : "Unable to communicate with database"
}
```
Required keys:
* status: Should always be set to "error".
* message: A meaningful, end-user-readable (or at the least log-worthy) message, explaining what went wrong.

Optional keys:
* code: A numeric code corresponding to the error, if applicable
* data: A generic container for any other information about the error, i.e. the conditions that caused the error, stack traces, etc.

## Whither HTTP? ##
But wait, you ask, doesn't HTTP already provide a way to communicate response statuses? Why yes, astute reader, it does. So how does the notion of indicating response status in the message body fit within the context of HTTP? Two things:

* The official HTTP spec has 41+ status codes, and there are many interpretations on how to use each one. JSend, on the other hand, defines a more constrained set of status codes, specifically related to handling JSON traffic in the context of a dynamic web UI.
* The spec is meant to be as small, constrained, and generally-applicable as possible. As such, it has to be somewhat self-contained. A common pattern for implementing JSON services is to load a JavaScript file which passes a JSON block into a user-specified callback. JSON-over-XHR handling in many JavaScript frameworks follows similar patterns. As such, the end-user (developer) never has a chance to access the HTTP response itself.

So where does that leave us? Accounting for deficiencies in the status quo does not negate the usefulness of HTTP compliance. Therefore it is advised that server-side developers use both: provide a JSend response body, and whatever HTTP header(s) are most appropriate to the corresponding body.

## License ##
The JSend specification (this page) is covered under a [modified BSD License](LICENSE.md)



