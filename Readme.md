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
        
        





3. Backend security

    Security not properly implemented can lead to financial loss. For our users and our business 

    Primary focus is on security vulnerabilites that occur because of our backend or the code we have written. 

    Different aspects of security
        1. Browser based security like cookies, https, cors etc 
        2. Network based security for tramission layer, http, encryption, compression etc.
        3. Server based security like attacks on the os 
        4. Backend based security the vulnerabilites we encounter due to the code or the application we write 


    One thing to notice is that as technology evolves and libraries are improved there will as security vulnerabilities but we have to put in place system and structure to help combat these vulnerabilites.

    There is no system that is truly secure.


    Thinking like the attacker 
        As a service or application developer we have to thing like that attackers of our system. 
        And how major question, attacker ask if where did the developer make an assumption.


        Our system becomes vulnerable to security breaches when we assume that

        1. The input from our users or frontend are clean
        2. Assume the users are who they say they are
        3. Assume request from our backend are coming from our frontend 
        4. Assume no know will open the network tab and modify network calls. 

        With an optimistic mindset, our system can easily get compromised.


    Categories of attackers on backend systems 
        1. Injection attacks 

            Are the type of attack where user inputs are malicious and our system treats them as code inside or raw data. 

            root cause
                Our backend application speaks multiple language. 
                It speaks SQL when interaction with the database
                - It speaks shell when it wants to use the command line 
                - It sends static files html,css,js with the browser. 
                - It speaks json when collecting inputs from the frontend.


            the issue is that some language will want to cross the boundaries into a different language 
            eg. json or html input make contain special character which means something in another language and will end up cross the boundaries to be executed as an sql code or a shell command, etc. 


            types of injection attacks 
                a. SQL injection 
                    A common type of attack where we have an sql statement, and we directly concatenate user's input to the sql statement.


                    eg we have an code that does this 

                    / 
                        String userEmail = req.body.email
                        String sql = "SELECT * FROM users WHERE email = '" + userEmail + "'";
                    /

                    we didnt think like an attacker, and made an assumption that the input from our user and frontend was clean, hence will directly concatenate the input. 


                    the user sends use "; DROP TABLE users; --- as their email

                    String userEmail = "'; DROP TABLE users; ---
                    String sql = "SELECT * FROM users WHERE email = '" "'; DROP TABLE users; --- "'";

                    the boundaries is crossed from users input that should be treated as data is now a code that delete our users table entirely. 


                    Prevention 
                        1. Parameterized queries: where we use function and apis from our database driver that treats user inputs as purely data instead of a sql command. 


                        / 
                            String sql = "SELECT * FROM users WHERE email = ?";
                            parameterized = db.execute (sql, useremail)

                        /

                        2. the use of modern database drivers and orms. sql injection is a classic vulnerabilites hence db driver implement some safety mechanism to compact all these vulnerabilites. We will be vulnerable if we bypass the security measures in place

                        3. Our database user has less privilege to perform destructive commands. 


                    The point is attacker can always come up with clever ways of performing attacker, we as backend engineer show always put the right mesures inplace to combat these attackers. 
                    Like input validation and not writing in a way that can treat user input as part of our code instead of treating it as data. 



                b. NoSQL injection attacks
                    Are no nosql database safe from injection attacks? No the same issue arise when user inputs contain special character which cross the boundaries of being data to a code that our application can execute


                c. Command injection 
                    works same as sql injection but attack target is OS instead of database like sql injection

                    eg a backend that accept user file and output file name. 

                    proccess the file with a command line tool like ffmpeg and produce an output file with the output file name. 

                    if a command looks like this 
                    
                    /
                        ffmpeg -h 120 -w 50 output_file_name;
                    / 
                    and we concatenate the file name directly to the command, we could have command injection is a user send malicious input as file name like ; rm -rf /;

                    ffmpeg -h 120 -w 50 ; rm -rf /;

                    this runs two commands and the second removes all files in our file system or os. 


                    prevents 
                        1. Seperate command from its argument.  most framework pass argument and runs them different to the process without them going to your shell interpreter

                        when user input is pass to the shell interpreter. it is treated as code

            All injection attacks happen when our system confused data with code. Whenever we are passing user input, we have to ensure the user input is purely data and should never be treated as something that can be executed or interpreted.


            Simple way is to use apis, functions, methodologies that enable as to seperate data from code. 

        2. Authentication

            Authentication the process of identifying the identity of a user. if we get it wrong, malicious users can impersonation other ursers and get access to their private data , take actions on their behalf and steal their money. 


            Rule of thumbs 
                If you can avoid building authentication by our self, we should avoid it and use an Authentication provider until we have the revenue to build our own auth from scratch or the bill from the auth provider is too huge. 

            The are many benefits of using an auth provider. 

            1. Authentication done well is complex. it is not stateless authentication that is using jwt token and verifying those token. Production system uses complex authentication which involves stateful authentication with things like 
    
             - sessions management like keep track of user sessions across multiple devices
             - revoking user sessions
             - linking a user login in with email password and later social auth as one account etc. 
             - it involves database, caches , timeouts 
             - storing session in distributed caches, databases
             - revoking sessions from caches, databases, 

            2. Authentication provider provide world class security. Auth providers are security companies that thinks about security vulnerabilites 24/7 hence by using them, we have access to world class security a

            3. Saves time : Authentication well done is complex and require a not of time to get it done. Authorization providers save us that time while we focus on our business logic. 


            We still need to understand how authentication works, what can go wrong and how to integrate it security. 


            a. Password storage

                1. Plain text password
                    Storing users password causes serious security breaks. 
                    
                    Breaches
                        Storing user password in plain text means that anyone 
                        
                        who gain access to our database whether is a developer, a database admin or attacker 

                        can impersonation the user in our system 
                        perform actions on their behalf
                        steal their money 
                        look into other system since users rarely use password manageer but uses one password across other system like the banks, social media, emails etc

                2. Hashing user password 

                    A better way to store user passwords is hashing it. Hashing is the process of taking the plain user password , passing it into hashing function with produces an output of fixed length

                    benefits of the hashing func are 
                        1. the output are of fixed length. any input provide fixed length output
                        2. it is one way: It is mathematically impossible to get the input back from the hashed output. 
                        3. same input should produce the same output


                    Breaches 
                        if our database gets breached by an attacker , or seen by employees, 
                        they cannot get the original password from the hashed value because hashes are built to be one way. It is mathematically impossible to get back the original input with the current computing power.

                        Rainbow tables 

                            What attacker do is they have a larger directory of common user password and their hashes called rainbow table and that is when they can map out the original password from the hash. 


                3. Hashing + salting user password
                    A way to combat the sameness of hashing function where the same input produces the same output which is prone to rainbow table attack is through salting. Salting is the process where before the input is passed to the hashing function, a random value is generated and added to the input first. 

                    make it same password producing different hashes. 
                    the salting value is stored 

                    breaches 
                        attackers have access to password hash and maybe the salt value 

                        but to produce the same hash. they still have to try billions of combinations to try. 

                    [to be continued, gap in knowledge]


                Industry standards for password hashing is argon2id (modern), brcypt.



        3. Sessions
            Session are ways to remember that the user has login in. Bring statefulness to our authentication. 

            A session should 
                1. Have a random identifier: The session identifier or id should be random because if an attacker guessing the session id, there can hijack the session of our user. 
                session id show be between 128 - 256 characters. making sure there are more possible session ids than atoms in the universe.


                2. Be stored in a datastore like our database or a distributed cache like redis. 
                    sessions are always stored in a datastore with some metadata like user info, timeouts, expiry, user agent,  like 

                3. Be sent to client browser and ask it to store it in a cookie so subsequent request the browser attached the cookie with the session id to them. 

                So a server will respond like this 

                HTTP/1.1 200 OK
                Set-Cookie: session_id=8f72a91c...; HttpOnly; Secure; SameSite=Lax

            Things to know about cookies 
                Configuation to be aware of in production systems 

    Summary 

        Backend security
        security vulnerabilites we introduce into our application as result of the code we write

        security vulnerabilites occur when we do not like attackers why ask questions like 
        where did developers of the system make an assumption

        assumption like 
            - the input from the user and frontend will be clean
            - our backend will be used only by our frontend
            etc


        types of security vulnerabilites 
            1. Injection attacks 
                our code treating data as code instead of purely being data.

                root cause: our backend speak multiple languages sql, shell, html etc. 
                    input from one language cross the boundary unto another language

                    SQL injection 
                        where in our code we directly concatenate user input to our sql query
                        we have sql query select * from user where email == userinput.email;
                        user sends as malicious input making our query change completely 
                        to another command or code 
                        eg select * from users where email == ""; drop table users; 

                        solution 
                            using parameterized queries
                            modern db driver combat common security vulnerabilites

                    Command injection 
                        where to directly concatenate user input into a command we run in our backend. 
                        like we using a image processing tool like ffmpeg, user sends us an image an image and an output name. 
                        user can send as a command like rm rf / wipe our entire file system os level attacks 

                        solution 
                            using the right language functions and tools 


                injection attacks are result of our code confusing data from code and we do always you the right apis, tool, methodologies to treat user inputs are purely data.

            2. Authentication 
                Issue we can introduce as a result of authentication 

                1. password storage 
                    1. plain text password storage 
                        where our users, password are stored in plain text 
                        anyone, employees attacker who gain access to our db can login as user, impersonating them, performing actions on their behalfs, stealing from them etc. 

                    2. store password hash 
                        before storing user password. it is password through a hashing func and the password hash is rather stored.

                        hashing func 
                            provide fixed length output not matter the size of the input
                            one way cant get the input from the hash output
                            same input will provide the same output

                        in a case of security breach
                            attacker have users email and password hash ,
                            they cannot get the input from the hash but they have a

                            random table 
                                a large directory that maps common passwords with their hashes 
                                looks up the table for hashes that matches ones in the db


                    3. store password hash + salting 
                        before hashing just the user password, we add the user password to a random cryptographically secure string and hash them do together 

                        this removes the issue of sameness because the string are random. two password which are the same with produce a direct output. 

                        rendering rainbow table useless. 

                        But attackers have a way around it 
                            as technology advance and we get access to more computer power 
                            like a single gpu core can compute billion password hashes in a second 
                            it will take a matter of some few days for attacker to get the original passwords from the hashes

                            attacker will do offline brute forcing. 

                        solution 
                            use industry standard hashing functions like bcrypt, argon2id
                            why 
                                they are built for password hash
                                has a cost factor where we specify how long a single hash should take 
                                eg 400ms 

                                attacker also will have to wait for 400ms to compute a hash, hence make the hashing long even with gpus. moving from days to decades


                            
                2. Stateful authentication / session based auth
                    when a user has proven their identity 
                    what happens is a session is created in the db with user metadata and some expiry

                    session id sent back to user to be store in a cookie
                    session id should be random to prevent attackers from guessing 

                    cookie sends the session id on subsequent requests. 


                    benefit 
                        if the user complain that their account is compromised, 
                        deleting the session from the db revokes the user in our system


                3. Stateless auth jwt based token
                    user proven identity

                    server put user info in payload and signs it with a secret key. 
                    sends the token back to user to be store safely in like a cookie not localstorage 
                    and to be sent on subsequent request. 

                    client how tells us how the user if from the payload 
                    if there is a tampering in the payload. signature verfies it 

                    issue 
                        revoking user session is impossible 
                        
                    solution 
                        blacklist token
                        short expiry times

                4. Cookie 

                    both auth recommend store their session id and jwt in cookies
                    but there are certain config for cookies that makes them robust and used in production systems

                    1. httponly: this flag prevent js from reading the cookie and protects us from xss 
                    attacks 

                    2. secure: tells browser to send the cookie only when using https because http traffic can be intersected. 

                    3. samesite with three options strict lax , none
                        samesite determines whether cookie should be sent cross site. 
                        strict one the same domain
                        lax sent cookie to only top level domains
                        none sents anywhere
                        


                5. brute force attack
                    rate limiting auth endpoints. being more restrictive than other endpoints 

                    implement layers of rate limit 
                        1. per ip 
                        2. per failed attempt on an attempt
                        3. global rate limit as a number of failed login in attempts


            3. Authorization
                user are in our system. what are the permitted to do or see?


        
                    






































    Browser based security like cookies and http
    Net
















Backend security



Security not paid attention to can cause distractive effect on my app. financially loss
security is a huge domain

different aspect of security 
browser security http cookies
network security transmission layer http encryption compression 
server security the os itself we deploy run in a process in the os

backend security all the vulnerabilites because of our app or code written in our backend

No app can be truly secured as technology involves and libraries evolve, there will always be security
vulnerabilites, but we have to try our best

How to think about attacker
they do not care about the framework or library

the ask this important question.
where did the developer made an assumption?

developer assumes the input from the user or the frontend will be clean 
assumed the user was who they say they was ?
or assume the request to their backend is coming from their frontend. 
assume no one will modified network calls 

Everytime i write i want to ask what could go wrong 


1. injection attacks 
all share the same the root cause

My backend speaks multiple languages at different context 
If wants to interact with Database, it speaks SQL
if backend sends static files it speaks html, css , js
if it want to open some files, system calls, it speaks sql 
etc. 

languages crosses other languages boundaries inside an ecosystem or browser, servers, databases, and tools




These languages has their own grammar and specific characters, this is where most vulnerabilites comes from 

vulnerabilites arise when a user cross the boundaries from one language to another. a user in browser using html, css, js input can mean a special command in a different language like sql, system calls that is the root cause of most of these injection based attacks 

a. SQL injection 
    whatever the user input it, we are embedding it in a string template in sql

    eg 

    String email;
    SELECT * FROM users WHERE email = '" + email + "';

    instead of sending 
    email = useremail@email.com
    password = userpassword 

    they send 
    email = "'; SELECT * FROM users; --

    the code above become 
    SELECT * FROM users WHERE email = '""'; SELECT * FROM users; -- "';


    sql injection has being a classic vulnerabilites. most of the modern driver has safety mechanisms which by defualt blocks
    back to back sql statements like 
        SELECT * FROM users WHERE email = '';DROP TABLE users; --';
        instend of the user send an input of email myemail@email.com
        they send ;DROP TABLE users; --
    alone the first statement runs 

    if old version of a db driver or one that do not have the security vulnerabilites the statement will work

    an attacker with correct skills can do alot of harm

        - use union statements extract data from our sensitive tables like payment information
        - db specific func to read files from server file systems
        - extract os command through the db 

    alot of attacker surface thats why sql injection is the most distractive attackers in decades.

    when we concatenate our sql string or template with user input. we are saving the input does not contain any special character

    Assume user input does not contain special characters

    prevention parameterized queries

    instead of doing string concatenation and pushing whatever is coming from the user. 

    SELECT * FROM users WHERE email = '" + email + "';

    const stmt = 'SELECT * FROM users WHERE email = $1';
    DB.query(stmt, userinput.email)
    the user input is treated purely as data or string it will not be confused as a command instead of code 


    validataion layer should have also throw an error that this does not look like string but a grabage email so it does not reach put to that point . 

    most db driver and orm support parameterized query. the only way to vulnerabilities is to deliberately bypass all the security set my the orms is building raw spring by ourself.

    what if i do not write SQL and use no sql or anytype of document database
        yes mongodb query object can also contain operators like not equal, exist, 

        from u take userinput input and passes it directly to the db queries attacker can inject these operators



Command injection
    works same as sql injection but attack target is OS instead of database like sql injection

    eg 

    a backend that accept user file and ouput file name, does some input processing using a command line tool like ffmpeg 
    when our handler calls 
        ffmpeg -h 120 -w 50 output_file
        attacker can send output_file_name as ffmpeg -h 120 -w 50 ; rm -rf /;
        and that will execute an a command wipe all our file system. 


    fit is to seperate command from its argument.  most framework pass argument and runs them different to the process without them going to your shell interpreter

    when user input is pass to the shell interpreter. it is treated as code


all injection attacks happen when our system confused data with code. when u use proper funs this can be prevented


whenever we are passing user input, 
we have to ensure the user input is purely data and should never be treated as something that can be executed or interpreted.

simplies way is to use apis, functions, methodologoes that enable me to seperate structure from the data. code from data 


Authentication
    the process verify the identity of the user. if we get auth wrong, malicious user can impersonation my users and access their private data, take actions on their behalf and also stealing their money.

    If i can avoid implementing auth myself. I should go for it . if i have the budget and approval of taking an auth provider 
    benefits 
        1. save a lot of time. 

            until our service scales to thousands of user and auth provider sending a billing or 10s of 1000s of $.
            most likely have a revenue source by that time.

        to get auth right , is not a simple jwt verifying, storing the token in cookie and verifies per each request there is not a lot of difficulty or weakness in that flow. 

        production systems is much more complicated than that and the scope is not clear
        general prefer stateful authentication, keep track of all the devices users have logined in , user or we the admins can revoke the session of all the devices at once. 

        if we want to implement stateful auth it gets more complex, not as simple as stateless 
        stateful auth involves databases, caches, timeouts , revoking from cache, database

        social logins oauth flow. email and password auth
            in the case where a user look in with social auth and later email, how do we link it as the same user. 


        Use auth provider 
            they are a security company that thinks of security 24/7 and they will provide world class security in our application
            until we have the revenue and human resource to implement auth ourselfs. 


    but we still need to understanding auth works , what can go wrong how to integrate securely

    1. Password storage: 
        storing users password in plan text can cause a breach and data being linked.
        user rarely use password managers. they use same password for every site
        developer and db admins can also see those passwords 

        hashing consist of hashing fun
            takes inputs of any kind of length, passes through a hashing func and return output of fixed length
            same input should produce same output
            it should be 1 ways we can get the same output from the same input
            but we should not be able to get the input from the output

        if the database is breached. attacker get hashs of user password. one major property of a hashing function is with the current computing power. no one can get the original password from the hash. 


        attacker build a huge storage of precomputed hases so i huge database with original passwords and their precomputed hashes.
        rainbow table is common password and their hashes 

        salting prevents this if the same input produces the same ouput  hash it is prone to rainbow table attackers
        to prevent the sameness , there is salting 

        long time bcrypt, recently argon2id is the industry standard 

        salting 
         user password, add randomly generate string ie salt before hashing prevent sameness and rainbow table 


        sessions
        we need to remember they are already authenticate
        when user login in 
            1. random session identifier, it has to be random because if they can guess it they can hijack any session of our users
            by making sure the session id is sometime between 128 and 256, making sure more possible session ids than number of atoms in our universe 

            2. storing session identifer in db like redis or primary db. in addition to session id we store 
                some kind of metadata like user info, when created (so we can timeit out ), expiry, user agents (what kind of devices user logined in)

            3. server sends session identifer to browser, ask browser to store in cookie 
                since its stored in a cookie all subsequent request to our backend
                get the session id , finds the user

                when using cookie, there are some config to be aware of to use in a production system
                    1. HttpOnly: if attacker is able to run js in users browser from my platform the my site is said to have xss vulnerabitlity. js can read cookie if only httponly is not set to true if storing sensitive data in cookie. 

                    2. Secure = true: browser will send the cookie back to my platform only if its https 
                    http traffic can be intercepted on public wifi 
                    sensitive cookie with session id or jwt token

                    3. same site. control if cookie can be sent throug a cors
                        strict 
                        lan
                        none 



 unless we have seoc
 scaling requirement
 prefer stateful authentication sessions when scaling requirements
 make distributed key value pair storage to store user sessions     
 session based authentication is always preferred and required 
 tradeoff of stateless auth is not worth it 
 session based auth, revoking strategies really straight forward , whole architecture is compatible with 
 most kinds of saas projects 

 if i use jwt use short expiratary time from minutes to hours not days
 always use httponly cookie than localstorage 



 rate limiting 

    attacker can try thousands of request per minute or different combination of authentication and password 
    on our auth endpoint since we are not limiting it all goes through

    2 things can happen
        1. find a match and get access to our server
        2. our server cant happen the load of all these requests per sec and server crashes 

    very important and mandatory security mechanism for auth endpoints and other endpoints 

    more restrictive in auth endpoint and more generous with other endpoints

    layered approach layers of ratelimit
        per ip based rate limiting , problems with ip address outgoing ip address from an university or org will match different user devices
        because all goes through same hub

        attacker use multiple ip address botnet, proxy networks, vpns
        do not prevent advanced attacks


        - per account limiting limit how many failed attempts that can happen for a single account 5 per 15mins lock account
            attacker try one password across different accounts

        - global rate limit. limit how many failed login attempt our system can take in a given timeframe. eg 100 failed attempts in a min
        when it happen system should raise an alert and flag that incident and now show catpcha to all user or block those ip address 

    prevent attacker getting access to a user through brute forcing 
    and servers crashing because of 1000s of requests attacker is sending 


Authorization issues
    after identity what can they do? what are their permissions ?

    Broken Object Level Authorization (BOLA)
        system does not check authorization of users on entities on db level but checks on routing layer not on repository.

        instead of books invoices, payment related info in our db they can download every invoice in our system. 
        get hold of all users financial data. modify and tamper with invoice urls 
        fix: do not just check at the routing level but also check in repository layer 

        also if invoice they throw an error msg of forbidden is the object level does not match the user. 
        technically it is right we have to throw these error with issue con

        if we return that we are confirming the object exists , distributive 
        attacker can finds all the invoice that exists in the system and they can go ahead and plan another layer of attack 
        eg social engineering. 

        we can do select * from where invoiceid = 7 and content.userid
        we will get no rows return 404 not found instead
        attacker can tell the different between something that exist or exist but belongs to someone else. 

        information leakage 


        Broken function level authorization (BFLA)
        restricting access to function than data.
        sensitive function only to be accessed by admins eg /admin/invoices

        there has to be another level where the there is an admin role checks 

        indirect object references
        broken access control patterns 

        using sequential ids in database 101, 102, 103 etc
        enable attacker to guess all the invoicees in our system. prone all kinds of attacks 
        using uuid


        categorize auth related attacks in 2 types.

            Horizontal auth attacks User A getting access to User B resourcees (BOLA Indorect object reference)
            Vertical auths (the user widen their scope. getting access to more of our system resources)


        Centralize auth related logic 
        Default deny 
        test authorization specifically 
        audit logs if a sensitve resource is access it should log that in the audit table , authorization checks fails we should know and take prevent measures 



Cross Site Scripting (XSS)
    malicious attacker get js to execute in a user's browser in the context of our frontend. 

    js scipt can read content of a page, make api request to remote servers as logged in user
    js has access to cookie (if not httponly ) and localstorage, redirect user to other phishing pages
    can also change the content of the page 


    root cause user defined content being treated as code than data



    xss is like injection attacks but happens in users browser

    user data inputted in html,js combined gets trated as code but mean for data
    prevention is sanitization.

    every time we are dealing with user provided content. search query html image
    we have to be every mindful as to how we are handling it and what kind of access we are giving 
    if that is done, we can get out of 99% of vulnerabilites 

CSP Content security policy

    provide by browser. 
    http header sent by server back to the browser telling the browser what kind of resources to execute or block
    like run script from these specific domains, do not run inline scripts 


Other security vulnerabilites 
    security misconfiguration. 
    push sensitive data to version control: pushing api keys, encrption secrets, database password to github etc 
    debug mode in production: logging mode is set at debug level in local , logging detailed error msgs, stack traces , databases sql query etc. 

        in production log level set to info



summarize 

    every vulnerabilities all about boundaries ie data crossing boundaries btw systems , privilage levels, programming languages, markup structures. we have to be every careful. 
    every crossed boundaries is where we encounter vulnerability like sql injection, untrusted input
    xss cross the boundaries of users ma
    bola , bfla crossed boundary of what different users should access 

    where is data crossing the boundary. what assumption am i making about the data 
    what if those assumptions are wrong. 



    no defense is perfect hence we need to think in layers 
        first 
        
        1. layer input validation
        2. parameterized operation, apis function that provide explicit parameteriz
        3. authorization checks verify every access at the point of access verified at the routing layer and the actually access was at repository layer. every at the point of access 
        4. security headers and policies cors, csp
        5. monitoring and logging: monitor suspicious activity to log properly 


    there can be weakness in one of these layers but if we have multiple layers an attacker has to bypass all this layers before they can cause any harm to our system
        












    






                        

                    
                    



























    the best authentication in production is stateful authentication instead of statelessness 













