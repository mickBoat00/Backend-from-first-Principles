Backend from principle 

This is my note learning backend from first principle. First principle means understanding the core concepts that is language, framework, library agnostic. 

Why 

Understanding a backend application from its core principles. concepts that stay the same regardless of the backend framework
Making me language agnostic where i can dive into any framework and understand what is happening at its core making me a valuable player in any team. 
Also makes me a system thinker, designing productive grade backends with ease. 


1. What is a backend, how request get to a backend and why do we need backend

A backend is a program on a computer, listening for http, grpc, websocket or any kind of request on a particular port, which clients connect to receive or send data. 

how does a request typically get to a backend 

Let say we have a fullstack application deployed on an AWS EC2 instance 

Flow
    -> User enters https://myapp.com/ in browser
    -> DNS Server or lookup for IP address
    -> AWS firewall (Security Group) 
    -> AWS EC2 instance 
    -> Reverse Proxy (Nginx) two server running eg Next js http://localhost:3000 & 
        spring boot http://localhost:8080
    -> Nginx decides / ==  http://localhost:3000 
    -> user browser receives HTML, then requests CSS, JS and starts building the DOM 
    -> While the JS executes it see we make a request to the same domain https://myapp.com/api/v1/products/ 

    fetch data from backend
    -> the browser cached the ip of the domain https://myapp.com 
    -> AWS Security Group firewall 
    -> AWS EC2 instance 
    -> Nginx decides /api =  http://localhost:8080
    -> User browser gets the data

    More details of the flow above ( collapsible )

        1. User enters the url of our app in their browser
        2. The browser does a DNS lookup of https://myapp.com 
            - in the DNS server there will be the domain and various records 
                A record points a domain or subdomain to an ipv4 address
                C NAME record points a domain to another domain 

                if the frontend is deployed in an EC2 instance
                /
                    myapp.com
                        A record           @            182.92.100.10 
                        A record           api          182.92.100.10
                        A record           www          182.92.100.10
                / 

                if the frontend was first deployed in vercel, and later a domain was added to prevent 
                using the default vercel domain

                /
                    myapp.com
                        A record           api          182.92.100.10
                        CNAME record       www          app.vercel.app
                / 
                
            We will use the former, where both frontend and backend are deployed on same EC2 instances

        3. Request is made to AWS networks for EC2 instance of 182.92.100.10 but before it reaches the instance
        there is an AWS firewall that will allow or prevent the traffic 
            / 
                AWS security group Inbound Rule 
                HTTPS TCP 443 0.0.0.0/0 - allows traffic 
            /
        4. Traffic is allowed and reaches the instance 182.92.100.10
        5. In the AWS EC2 i have two app running a frontend on http://localhost:3000 and 
            backend http://localhost:8080, hence we need a reverse proxy to determine where the requests should go 

        6. We have an nginx in the EC2 that will forward traffic because it has been configured to know what to do 
            nginx.conf

            / 
                server {

                    location / {
                        proxy_pass http://localhost:3000;
                    }

                    location /api {
                        proxy_pass http://localhost:8080;
                    }

                    listen 443 ssl;

                }
                
            /

        7. Browser receives the data and continues painting. 

Why do we need a backend and not have everything in the browser

1. Security: When a user's visit a website , the browser fetch html,css,js from the server and is executed by the browser in the users' computer.The browser is the runtime. the browser are often sandoxed
ie isolated from the os process, file system the code can access limited amount of resources

this prevent user browser running codes that can access files or data from the users computer. often times the backend need access to the underlying file system,access env variable etc.browser do not allow that make it a huge restriction for a backend server.

2. CORS restricted backend usually calls external apis more. with a browser, it can only call external apis which has the appropriate cors headers, limited complex logic for applications. 

3. database browser cant establish persistent connection to a database. 

4. Computer power: Frontend application exist everywhere. the user might a less computer power device to perform complex business which can cause a lag or device of the application. 






2. HTTP Protocol 
Where it all begins 

When a user visits a website in their browser, the browser requests a webpage from a computer somewhere on the internet and displays it for the user. 
Computers on the internet are identified by IP address, hence the browser resolves the domain eg http://myfrontend.com to an IP address. I previously understood that the domain myfrontend.com on the DNS server has an A record that points a domain name to an IP address. 

That computer with the webpage is a server because it is a computer which clients eg the user's browser or an application request resources from. 

Now that the IP address is found, there has to be a connection before the client and server can communicate. 

There are various ways clients and servers communicate and one of the most common is a protocol called HTTP HyperText Transfer Protocol. 


It is a standardized way of communication between clients and servers. HTTP uses TCP as its underlying protocol for client server connection because before any communication has to take place, a connection has to be established.


Core ideas in HTTP 

HTTP has 2 main ideas at its heart. 

1. Statelessness: When a client and server are communicating, that is sending messages in the form of HTTP requests and receiving a response back from the server as an HTTP response, servers do not remember past requests the client sent or responded to, hence the client has to send all the necessary information in the request to get the desired response. 

eg Client sends login credentials in a request and receives a response of successful login. 
Subsequent requests after that, if the client does not send the necessary information in the form of a cookie or token, the server will not remember that the client is logged in.

Stateless servers need no per-client memory. Requests can hit any machine.



2. Client server model: With HTTP Protocol the client always initiates the communication. Servers do not initiate communications


HTTP versions
Over the years HTTP evolved to improve client server interaction
Starting from 

HTTP/1.0: The TCP connection is closed after every request response cycle. Leading to latency and being resource intensive. 

HTTP/1.1: The same TCP connection can be used to send multiple requests and receive multiple responses. 

HTTP/2: Multiplexing. HTTP/1.1 had an issue called head of line blocking where until the 
initial request has been responded to, all the current requests will have to wait. HTTP/2 sends many requests on the same TCP connection so one slow response does not block the others at the HTTP layer.

HTTP/3: HTTP/2 still has TCP-level head of line blocking. HTTP/3 uses QUIC over UDP as its underlying protocol for communication. 

HTTP/1.1 is still widely used today. 

A typical HTTP request has an HTTP method, a URL, HTTP version, HTTP headers, and an optional body.
A typical HTTP response has an HTTP version, a status code, HTTP headers, and an optional body.

HTTP request example 

PUT /api/users/12 HTTP/1.1
Host: backend.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json

{
    "name": "Ama Mensa"
}

HTTP response example

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 50

{
    "msg": "User updated"
}



HTTP Methods
HTTP methods are verbs which show the action the client wants the server to perform on a particular resource

GET - get a resource from the server
POST - create a resource on the server
PUT - update and completely replace a resource on the server
PATCH - update certain parts of a resource on the server
DELETE - delete a resource from the server
OPTIONS - used in CORS preflight request 


Idempotent
    Idempotent means retry a request or send the same request multiple times will have the same effect on the resource
    Idempotent methods GET, PUT, DELETE

Non idempotent 
    Non idempotent means retry a request or sending a request multiple times will have a different effect on the resource
    Two POSTs can both return 201 and still create two rows.
    Non idempotent method POST


HTTP status codes 
Quickly communicate the result of a request in a standardized way. Standardized means consistency between how a server should communicate the results of the request. There should not be a case where the status code is telling a different msg from the body. 

This standard is what our servers should follow. 

1xx - information
2xx - success
3xx - redirect
4xx - client error
5xx - server error 

1xx information 
used for information to the clients. 
Can be used in a situation where the request is huge when the headers are sent before the body so 

100 Continue - in a situation where the request is huge and the headers are sent before the body so 100 indicates headers received, continue with the body

101 Switching Protocol - Where the communication protocol for the url has changed from HTTP to websocket. 


2xx - success response
200 OK - request was successful and server sending requested resources or performing the requested action
201 CREATED - successful creating a new resource
204 NO CONTENT - successful but no content

3xx - redirection request
301 MOVED PERMANENTLY - requested resource has been permanently moved to a new url.
302 MOVED temporarily - for now moved to a temporarily url
304 NOT MODIFIED - used in caching

4xx - client error
400 BAD REQUEST - client sends invalid data
401 UNAUTHORIZED - request requires valid authentication
403 FORBIDDEN - refused to authorize the request
404 NOT FOUND - resource that is not available
405 METHOD NOT ALLOWED - this method is not allowed on this URL
409 CONFLICT - when resources are not unique 
429 TOO MANY REQUEST - rate limit client request


5xx - Server error 
500 Internal Server Error - something broken or exception was raised which was not handled.
501 Not implemented - this server does not implement that method. 405 is this method is not allowed on this URL.
502 Bad gateway - usually happened by proxies and load balancers, upstream invalid response
503 Service Unavailable - Server is not available or too busy to handle more requests.
504 gateway timeout - the upstream server failed to respond in time, nginx did not receive a response from original server



HTTP headers 
Are metadata which are key value pairs of an HTTP message. they provide certain information about the HTTP message like the content-type, date of the message 
security features. When sending a package think of HTTP headers as the info that is pasted on the package like where the package is going to etc.

categories of headers 
request headers - used to understand client env, preference and capabilities- eg User-Agent, Authorization, Accept, Cookie etc

general header - general info about the HTTP message - eg Date, Cache-Control, Connection etc.

representation header - info about the HTTP message's body ensure client, server knows how to interpret them - eg content-length, content-encoding, etag

security headers - enhance the security of the http request/response- eg Strict-Transport-Security(HSTS), Content-Security-Policy(CSP), Set-Cookie


Importance of HTTP header

Extensible: HTTP header makes HTTP extensible because it gives room to easily add custom headers for new features or security enhance without altering the underlying protocol

Remote Control: Headers act as remote control, where clients can influence or control the kind of behaviour they want from the server eg. client wants a response in spanish instead of english with Accept-Language: es header



HTTP Caching
Storing copies of responses for reuse. Reducing the need for repeated request to the server, reduces loadtime, bandwidth, server loads.

The client does not need to download a lot of data and server does not need to send a lot if the data has not changed.


eg of a request response with caching

GET /api/products HTTP/1.1
Host: backend.com

Response

HTTP/1.1 200 OK
Cache-Control: max-age=10, public
ETag: 3142
Last-Modified: Thu 17 Sep 2026 19:09:10 GMT

{
    "name": "product1"
}

Cache-Control is the header that tells the client to cache the response for a period of 10s as specified by max-age=10.
While the cache is fresh, the client does not send another request. It uses the stored copy.

ETag is the hash value of the response which tells us that if the etag changed the response has also changed

Last-Modified as the name implies, the last time the response was modified


After the cache is stale, the client may send the same request with validators

GET /api/products HTTP/1.1
Host: backend.com
If-None-Match: 3142
If-Modified-Since: Thu 17 Sep 2026 19:09:10 GMT


the server responds with the response has not changed 

HTTP/1.1 304 Not Modified
Cache-Control: max-age=10, public

or it will respond with the new data, if the response has changed.

HTTP/1.1 200 OK
Cache-Control: max-age=10, public
ETag: 2398
Last-Modified: Sat 19 Sep 2026 18:00:10 GMT

{
    "name": "product1 updated"
}

Less data is sent when cached. 

Content Negotiation
The mechanism client and server agree on the format to exchange data. Clients can indicate their preferred format and server can respond with the format or a fallback format. 

GET /products HTTP/1.1
HOST: backend.com
Accept-Language: es
Accept: application/json
Accept-Encoding: gzip, deflate, br, zstd


server responds appropriately. in this case responds in spanish because of the header Accept-Language: es


Transferring of large files (image, audio, video)

A JSON body is small. Images, audio, video are not.

Client to server. Multipart.
multipart/form-data is used to send files from the client to the server.
The body is split into parts. That is why it is called multipart.

POST /api/upload HTTP/1.1
Host: backend.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxk

------WebKitFormBoundary7MA4YWxk
Content-Disposition: form-data; name="file"; filename="shoe.jpg"
Content-Type: image/jpeg

<binary image bytes>
------WebKitFormBoundary7MA4YWxk--

Server to client. Chunked stream.

GET /videos/intro.mp4 HTTP/1.1
Host: backend.com
Accept: video/mp4

HTTP/1.1 200 OK
Content-Type: video/mp4
Transfer-Encoding: chunked
Connection: keep-alive

<chunk of video bytes>
<chunk of video bytes>
...

chunked means the file is sent in pieces. The connection stays open until the last chunk.
keep-alive means after the file, do not close the TCP connection.


SSL, TLS, HTTPS

SSL original protocol for secure communication between client web browser and server.
encrypt data so password and credit card number cannot be intercepted by attackers 
SSL is outdated due to security vulnerabilities

TLS modern method used by client and server for data transmission   
TLS encrypts data in transit by using certificates to authenticate the server and establish encrypted connection prevent eavesdropping and data breaches.

HTTPS is HTTP over TLS. SSL is the old name.
When a user visits a website with https, TLS encrypts the communication between your browser and server
prevent sensitive data like login from being intercepted.



Simple HTTP messages flow between users browser and our deployed fullstack application on AWS 

We have a fullstack ecommerce application deployed on two EC2 instances with two domains
1. AWS EC2 1 contains our react + vite application Domain https://myfrontend.com
2. AWS EC2 2 contains our backend application Domain https://mybackend.com


Flow 
    1. User visits our website in their browser https://myfrontend.com/
        - this tells the browser, communicate with the website myfrontend.com using https protocol. 
        * https is HTTP over TLS where the messages are encrypted. For this flow we will use http to understand the flow

    2. browser looks up the ip address of myfrontend.com through a DNS server which returns the A records that point to an ip address. 

    3. Before http communication can happen the user browser needs to establish TCP connection with the server. 
        - The user browser and the EC2 1 instance does the 3 way handshake (SYN, SYN-ACK, ACK) and connection is established.

    4. Browser sends first message HTTP request
        GET / HTTP/1.1
        Host: myfrontend.com
        Accept: text/html 
        Connection: keep-alive

    5. Our frontend EC2 instance responds with
        HTTP/1.1 200 OK
        Content-Type: text/html
        Connection: keep-alive

        <!DOCTYPE html>
        <html>
        <head>
            <title>My App</title>
            <link rel="stylesheet" href="/app.css">
        </head>
        <body>
            <div id="root"></div>
            <script src="/app.js"></script>
        </body>
        </html> 

    6. While the user's browser parses the html, it sees links to css and js and makes request for them over the same tcp connection
    <link rel="stylesheet" href="/app.css">

    GET /app.css HTTP/1.1
    Host: myfrontend.com
    Connection: keep-alive

    frontend server responds with 
    HTTP/1.1 200 OK
    Content-Type: text/css
    Content-Length: 156
    Connection: keep-alive

    body {
        margin: 0;
        font-family: Arial, sans-serif;
    }
    ...

    For js 
    GET /app.js HTTP/1.1
    Host: myfrontend.com
    Accept: */*
    Connection: keep-alive

    Frontend server responds with 
    HTTP/1.1 200 OK
    Content-Type: application/javascript
    Content-Length: 85000
    Connection: keep-alive

    const response = await fetch("https://mybackend.com/api/v1/products");

    executes the js. Now lets notice that in the js there is a call to our backend in a different domain.


    7. User browser establish a new TCP connection with our backend server AWS EC2 2 instances and sends the first message HTTP GET request

    GET /api/v1/products HTTP/1.1 
    Host: mybackend.com 
    Origin: https://myfrontend.com
    Accept: application/json
    Connection: keep-alive

    Notice the header Origin: https://myfrontend.com. it was added by the browser because it has a strict same origin policy. CORS is discussed below.

    Backend server EC2 instance 2 responds with

    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: 100
    Access-Control-Allow-Origin: https://myfrontend.com

    {
        "data": [
            {
                "name": "product1"
            }
        ]
    }

    8. Js gets the list of products and displays it on the html. 



CORS (Cross Origin Resource Sharing)    
    By default browsers have a same origin policy. For a simple cross origin request the browser still sends the request. It hides the response from JS if Access-Control-Allow-Origin does not match.

    Notice from the flow above, on step 7, the js from the domain myfrontend.com, made a request for products from mybackend.com which is a different domain. The browser noticing that added an additional header to the request headers. 

    Origin: https://myfrontend.com  

    before making the request. 

    And when the response comes from the server, the browser also checks the Access-Control-Allow-Origin header in the response, and if the page's origin is not included like Access-Control-Allow-Origin: https://myfrontend.com OR Access-Control-Allow-Origin: * , the browser blocks the js from getting access to the response. 


    There are two types of request that are made with CORS
    1. Simple request are request made without a preflight OPTIONS request.

    2. Preflight are required to enquire the capabilities of the different domain's server

    There are a few things the browser checks to make a preflight request. They are as follows:
        1.  The HTTP method is not GET, POST, HEAD. 
            ( PUT & DELETE ) requests send a preflight request of OPTIONS before the actual request.

        OR 

        2. The request includes non simple headers like Authorization or X-Custom-Header

        OR

        3. The request has content-type other than application/x-www-form-urlencoded, multipart/form-data or text/plain


        when an HTTP request to a different domain meets any of the criteria above, the browser sends a preflight request which is an HTTP request with an OPTIONS method to enquire the capabilities of the server. 


        For eg a request made by the frontend to the backend to 
        update a user's order. like this 
        
        PATCH /api/order/1 HTTP/1.1
        Host: mybackend.com
        Origin: https://myfrontend.com
        Content-Type: application/json
        Authorization: Bearer eyuid...

        {
            "status": "pending"
        }
        
        will trigger a preflight request because all the conditions were met.


        OPTIONS /api/order/1 HTTP/1.1
        Host: mybackend.com
        Origin: https://myfrontend.com
        Access-Control-Request-Method: PATCH
        Access-Control-Request-Headers: Authorization, Content-Type


        Server response
        HTTP/1.1 204 No Content
        Access-Control-Allow-Origin: https://myfrontend.com
        Access-Control-Allow-Methods: PATCH, PUT, DELETE #server says i allow these methods on that url
        Access-Control-Allow-Headers: Authorization, Content-Type
        Access-Control-Max-Age: 86400 #server says this response will be the same for the next 86400 so dont make another request again 
         
        
        before the original PATCH request will be sent a response comes back
        HTTP response 

        HTTP/1.1 200 OK
        Access-Control-Allow-Origin: https://myfrontend.com
        Content-Type: application/json

        {
            "msg": "Order updated"
        }
        





















2. HTTP Protocol

Where is it all starts 

There are various ways client like a browser and a server communicate. One of commmon ways is through HTTP protocol. 
Http protocol is a standardized way on which a client and a server should communicate. 

There are two main ideas that are the heart of http protocol 
1. Stateless: Server do not keep memory of past interactions with clients, this means client should send all the neccessary information in their request to the server. eg authentication token, session information, cookies etc.

benefits of stateless
    simplicity server does not need to store session info 
    scalability makes it easy to distribute request across multiple servers. because no single server need to keep track of a session 

Because http is stateless developer develop state management techniques like cookies, sessions, tokens 

2. client server models 
client is a browser or application which initiates the communication by sending request to a server 
client is responsible for provide all the info, the url, headers, methods, body etc.

server is a application waiting for a request from the client, process and sends back resources like webpage, data json, text file or any content

Http protocol communication is always initiased by client to get some kind of response from the server.
https is more secure version of http 


To send a request and recieve a response. A connection has to be establish between the client and server. http uses tcp

OSI model is referred to when talk about sending and recieve data over the network 
backend enginner often deals with the application layer. tcp handshake, tls encryption. 


There are being different various of http through the years. defining how client and server send data 
HTTP 1.0 each request opens a new connection, connection has to be established for every request and response, which slowed performance
HTTP 1.1 persistent connection allow, multiple request response over the same connection
HTTP 2 multiplexing 


client and server establish a network connection and messages are sent and recieves 

Request message sent by the client

PUT /api/users/12 HTTP/1.1
Host: backend.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer eyJhbGciOi...

{
    "name": "Ama Mensa"
}

http method
resouce url 
http version
http headers 
http body

Response message 

HTTP/1.1 200 OK
Content-type: application/json
Content-Length: 50

{
    "msg": "User updated"
}

http version
http status code 
http status code value 

Http header 
key value of different metadata about the request and response. 

think of a package, it has certain info about the package before we know the main contents of the package
this gives us a way to quickly know certain information about the package


categories of headers
request headers eg User-Agent, Authorization, Accept, Cookie, helps server understand client env, preference, capabilites
general headers eg date, cache-control, connection some message about the message
representation headers content-type, content-length, content-encoding, etag provide some info about the body of the request / response
ensure client and server how to intepret the request or response. 


Security headers Strict-Transport-Security(HSTS), Content-Security-Policy(CSP), Set-Cookie, X-Frame-Options enhance the security of the request and response

http header makes http extensible because headers can be easily added or customized without altering the underlying protocol
http header acts as remote control to there server by influencing the kind of behaviour the server should give back


http methods 
method represents the different kinds of action that the client can request on the server. 
method defines the intent of the interaction 
GET - get a resource from the server
POST - create a resource on the server 
PATCH - update some part of a resource, partially update some parts of a resource
PUT - also update resource but the data in the request should completely replace the resource. 
DELETE - remove a resource
OPTIONS - used in CORS flow, used by browser preflight to fetch the capabilites of the server

Idempotent
    two or more request of the same kind (header and body) return in the same kind of response
    eg send the same GET request will return the same response all the time.
    Idempotent methods GET,PATCH,PUT,DELETE

Non Idempotent
    two or more request of the same kind, returns different response.
    eg two POST request with the header and body returns different result,
    Non Idempotent POST

CORS 
    by default browsers have a same origin browser take restricted domains from make requests to domains that is not theirs.
    for security. 
    This allows server to specify who can access their requests and how. 

    types of request with CORS 
    
        1. Simple request
        imagine client at https://meichel.com
        backend at https://api.meichel.com

        two different domains meichel.com, api.meichel.com

        eg 
        user opens https://meichel.com the JS requests data from the backend
        
        /GET /api/v1/products HTTP/1.1
        Host: https://api.meichel.com 
        Origin: https://meichel.com - browser adds it automatically since there are different domains
        Accept: application/json

        it request gets to the server. checks its CORS and if the Origin is allowed 
        adds Access-Control-Allow-Origin to the header and sends the response. 

        HTTP/1.1 200 OK
        Content-type: application/json
        Access-Control-Allow-Origin: https://meichel.com

        {
            "name": "product1"
        }
        


        Once the browser see the with the origin in the values the response to allow to the frontend 
        else it blocks the frontend from getting access to the data.




      2. Preflight Request 
      Preflight request happens when any of this is true

        1. The http method is not GET, POST, HEAD. 
            (PUT & DELETE) sends a preflight request of OPTIONS before the actual request

        2. The request includes non simple headers like (Authorization, X-Custom-Header)

        3. The request has content type other than application/x-www-form-urlencoded, multipart/form-data or text/plain


        when any of these conditions are true. the browsers sends an OPTION request to enquires some capabilities of the server


        eg 

            OPTIONS /api/v1/products HTTP/1.1
            Host:  https://api.meichel.com 
            Origin:  https://meichel.com
            Access-Control-Request-Method: PUT
            Access-Control-Request-Headers: Authorization


            if the server supports cross origin request. COR
            it will respond with and if it does not handle CORS it will not respond with it and the request will be blocked. 


            
            HTTP/1.1 204 No Content
            Access-Control-Allow-Origin:  https://meichel.com
            Access-Control-Allow-Method: PUT, DELETE #server says i allow these methods
            Access-Control-Allow-Headers: Authorization 
            Access-Control-Max-Age: 86400 #server says this response will be the same for the next 86400 so dont make another request again 
            to me about it. 


HTTP status codes 
Quickly communicate the result of a request in a standardized way
Standardized enable consistency with how server communicate with different clients. 
if a request was successful your server should return 201, this are standards what we have to follow

1xx - information
2xx - success
3xx - redirect
4xx - client error
5xx - server error 

Response code from the server with 1xx indicates headers has been recieved
can proceed to send the request body

Used when client sends request headers first
used when server tells the clients its switch protocols say from http to websocket

2xx - success response
200 OK - request was success and server sending request resources or performing the requested action
201 CREATED - successful creating a new resource
204 NO CONTENT - successful but no content


3xx - redirection request
301 MOVED PERMANENTLY - request resource has been permenantly moved to a new url.
302 MOVED temporariy - for now moved to atemporarily url
304 NOT MODIFIED 

4xx - client error
400 BAD REQUEST - client sends invalid data
401 UNAUTHORIZED - request require valid authentication
403 FORBIDDEN - refused to authorize the request
404 NOT FOUND - resource that is not available
405 METHOD NOT ALLOWED - invalid http method used
409 CONFLICT - when resource are not unique 
429 TOO MANY REQUEST - rate limit client request


5xx - Server error 
500 INTER- something broken or exception was raised which was not handled.
501  NOT IMPLEMENTED - does not support the http method but plans to add it in the futue
502 bad gateway - usually happended by proxies and load balancers, upstream invalid response
503 service unanavilable -
504 gateway timeout - the upstream server failed to respond in time, nginx did not recieve respond from original server


HTTP Caching
Storing copies of responses for reuse. Reducing the next for repeated request to the server.
reduces loadtime, bandwith, server loads
the client do not need to download alot of data and server do not need to send alot if the data has not changed.


Initial request

GET /api/products HTTP/1.1
Host: backend.com


HTTP/1.1 200 OK
Cache-Control: max-age=10 , public
Etag: 3142
Last-Modified: Thu 17 Sep 2026 19:09:10 GMT

{
    "name": "product1"
}

No fetch for the same resource another get 

GET /api/products HTTP/1.1
Host: backend.com
If-None-Match: 3142
If-Modified-Since: Thu 17 Sep 2026 19:09:10 GMT

the 2 header 
If-None-Match: 3142
if the etag, ie hashed version of the response object has not change from 3142
or if the request has been modified after the modified since, then send us the updated resource 
otherwise i will used the cached version


HTTP/1.1 304 Not Modified
Cache-Control: max-age=10 , public


if the resource is modified after that and client sends the say request
Server reponse with the new data

HTTP/1.1 200 OK
Cache-Control: max-age=10 , public
Etag: 2398
Last-Modified: Sat 19 Sep 2026 18:00:10 GMT

{
    "name": "product1 updated"
}

better solution for caching eg react query client side caching. client has power over when to use a cache version 
or when it wants to refetch


Content Negotiation
Where client and servers exchange information about different types encoding representation of the content
the mechansim client and server agree on the format to exchange data.
client can indicate its preferered format and server can respond with the format or a fallback format. 

GET /products HTTP/1.1
HOST: backend.com
Accept-language: es
Accept: application/json
Accept-Encoding: gzip, deflate, br, zstd

Each request response cycle require new connection
establishing connection is resource intensive and slow. 
keep-alive header maintain persistence connection, 
allow client and servers to use the same connection for multiple request response until one of them decide to close it. 


reduce latency and save resources as few connections needs to be established
explicit tells the connection to keep alive with variable like how long the connection should be open with a timeout
and how many request we can send before the connection is closed. max-value 
explicitly closed 


Handling large request and response 
sever takes large video audio file anytime larger than json data
and how client can recieve large responses 

client sends large request to the server
multipart is used to send large files to the server form client
the binary data of the file is transferred in different parts that is why it is called multipart


transfer large file from a server to client 
in streams with headers
Content-Type: text/event-stream
Connection: keep-alive





TLS modern method used by client and server for data transmission   
encrypt data in transit
tls use certificates to authenticate the server and establish encrpted connection prevent eavesdropping and data 
breaches

HTTP more secure https provides by tls or ssl
visits a website with https, ttls encrpts the communicate between your browser and server
prevent sensitive data like login from being intercerpted

visualize the whole flow.



Routing 
http methods describe the intennt of your action to perform on a server.
routing express the where the intent should be applied

GET /api/products HTTP/1.1

Client is saying a i want to get some data from where? /api/products 
routing is many url params to a server side logic 
types of routing 
1. Static routes eg /api/products, static root dont have variable params in the route
2. Dynamic routes eg /api/product/:id  dynamic root has variable param in the route
3. query params eg /api/products?search=shoe GET request do not not contain body when to send request to the server 
4. nested routs eg /api/products/:id/reviews/:id
get do not have some 







HTTP Statefulness 
If a login in request 
We expect the server to remind that i looked in 
but without the proper stateful mechanisms HTTP  will not remind that you logined in a previous request 
when 