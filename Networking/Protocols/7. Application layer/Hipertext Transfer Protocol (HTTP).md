Is the foundation of the World Wide Web, and is used to load webpages using hypertext links. HTTP is an application layer protocol designed to transfer information between networked devices and runs on top of other layers of the network protocol stack. A typical flow over HTTP involves a client machine making a request to a server, which then sends a response message.

### HTTP request?
An HTTP request is the way internet communications platforms such as web browsers ask for the information they need to load a website.
Each HTTP request made across the Internet carries with it a series of encoded data that carries different types of information. A typical HTTP request contains:
- HTTP version type.
- a URL
- HTTP request headers
- Optional HTTP body

### HTTP method?
An HTTP method, sometimes referred to as an HTTP verb, indicates the action that the HTTP request expects from the queried server. For example two of the most common HTTP methods are **GET** and **POST**; a GET request expects information back in return (usually in the form of website), while POST request typically indicates that the client is submitting information to the web server (such as form information, e.g. a submitted username and password).

### HTTP request headers?
HTTP headers contain text information stored in key-value pairs, and they are included in every HTTP request (and response). these headers communicate core information, such as what browser the client is using and what data is being requested.

### HTTP request body?
The body of a request is the part that contains the 'body' of information the request is transferring. The body of an HTTP request contains any information being submitted to the web server, such as username and password, or any other data entered into a form.

### HTTP response?
An HTTP response is what web clients (often browsers) receive from an internet server in answer to an HTTP request. These responses communicate valuable information based on what was asked for in the HTTP request.
A typical HTTP response contains:
- an HTTP status code.
- HTTP response headers
- optional HTTP body

### Status code
HTTP status codes are 3-digit codes most often used to indicate whether an HTTP request has been successfully completed. Status codes are broken into the following 5 blocks:
1. 1xx Informational
2. 2xx Success
3. 3xx redirection
4. 4xx Client Error
5. 5xx Server Error
The "xx" refers to different numbers between 00 and 99.

### HTTP response headers?
Much like an HTTP request, and HTTP response comes with headers that convey important information such as the language and format of the data being sent in the response body.

### HTTP response body?
Successful HTTP response to GET request generally have a body which contains the requested information. In most web request, this is HTML data that a web browser will translate into a webpage

https://www.cloudflare.com/learning/ddos/glossary/hypertext-transfer-protocol-http/