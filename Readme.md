# Backend from first principles

## Table of contents

1. [Introduction](#1-introduction)
    a. [Why backend from first principles](#a-why-backend-from-first-principles)
2. [What is a backend, how a request gets to the backend, and why we need backends](#2-what-is-a-backend-how-a-request-gets-to-the-backend-and-why-we-need-backends)
    a. [What is a backend?](#a-what-is-a-backend)
    b. [How does a request get to your backend server from a user's browser](#b-how-does-a-request-get-to-your-backend-server-from-a-users-browser)
    c. [Why do we need a backend, and can't we do everything on the frontend?](#c-why-do-we-need-a-backend-and-cant-we-do-everything-on-the-frontend)

3. [HTTP Protocol](#3-http-protocol)
    i. [Two ideas at the heart of HTTP](#i-two-ideas-at-the-heart-of-http)
    ii. [HTTP Versions](#ii-http-versions)
    iii. [HTTP message](#iii-http-message)
    iv. [HTTP Headers](#iv-http-headers)
        a. [Why headers](#a-why-headers)
        b. [HTTP header types](#b-http-header-types)
        c. [Benefits of headers](#c-benefits-of-headers)
    v. [HTTP methods](#v-http-methods)
        a. [Idempotent vs Non Idempotent](#a-idempotent-vs-non-idempotent)
        b. [Cross Origin Resource Sharing (CORS)](#b-cross-origin-resource-sharing-cors)
            i. [Simple request](#i-simple-request)
            ii. [Preflight request](#ii-preflight-request)
    vi. [HTTP response code](#vi-http-response-code)
    vii. [HTTP caching](#vii-http-caching)
    viii. [HTTP content negotiation](#viii-http-content-negotiation)
    ix. [HTTP based compression](#ix-http-based-compression)
    x. [Handling large request and response](#x-handling-large-request-and-response)

4. [Routing](#4-routing)
    i. [Types of routing](#i-types-of-routing)
        a. [Static routes](#a-static-routes)
        b. [Dynamic route](#b-dynamic-route)
        c. [Query parameter](#c-query-parameter)
        d. [Nested route](#d-nested-route)
        e. [Route versioning and deprecation](#e-route-versioning-and-deprecation)
        f. [Catch all route](#f-catch-all-route)
5. [Serialization and Deserialization](#5-serialization-and-deserialization)
    i. [The language barrier](#i-the-language-barrier)
    ii. [Solution](#ii-solution)
    iii. [The flow](#iii-the-flow)
    iv. [OSI model](#iv-osi-model)
    v. [Serialization standards](#v-serialization-standards)

---

<h2 id="1-introduction"><u>1. Introduction</u></h2>
<div style="margin-left: 2em;">

<p>Backend is not just about building CRUD APIs. To me, backend is about building reliable, scalable, fault tolerant, maintainable and efficient systems.</p>

<p>But how do we learn to build such systems? There are a lot of concepts concerning backend development on the internet, but how do we understand the big picture? When and how do these concepts come together to build such systems?</p>

<p>Secondly, we usually learn backend from a programming language or a framework's point of view, but how much of this knowledge can we transfer when we are to work with other frameworks?</p>

<p>Hence these notes cover the foundational concepts that are abstracted by the languages and frameworks we use every day, and how these foundational concepts come together to build a production grade backend.</p>

<h3 id="a-why-backend-from-first-principles">a. Why backend from first principles</h3>
<div style="margin-left: 2em;">

<ol>
<li>Understanding the core concepts that remain the same across every backend system</li>
<li>Elevate me from a framework developer to a true software engineer</li>
<li>Get up to speed quickly when asked to build with a different language / framework</li>
<li>Understanding the core concepts helps me choose the right tool for the right job, whether choosing a programming language, or using DB search against Elasticsearch, Redis for caching</li>
</ol>

</div>
</div>

---

<h2 id="2-what-is-a-backend-how-a-request-gets-to-the-backend-and-why-we-need-backends"><u>2. What is a backend, how a request gets to the backend, and why we need backends</u></h2>
<div style="margin-left: 2em;">

<h3 id="a-what-is-a-backend">a. What is a backend?</h3>
<div style="margin-left: 2em;">

<p>A backend is an application on a computer on the internet that is listening for a request — whether HTTP, gRPC, WebSocket, or any kind of request — through an open port, and sends a response.</p>

<p>It can be termed a server because it serves some kind of content, like static files (HTML, CSS, JS) or data in JSON.</p>

</div>

<h3 id="b-how-does-a-request-get-to-your-backend-server-from-a-users-browser">b. How does a request get to your backend server from a user's browser</h3>
<div style="margin-left: 2em;">

<ol>
<li>The browser needs to find the IP address of our backend server from our domain name.</li>
<li>It makes a request to a DNS server for our domain. Our domain contains different types of records:
    <ul>
    <li><strong>A record</strong> — maps a domain / subdomain to an IP address</li>
    <li><strong>CNAME record</strong> — maps a domain to another domain</li>
    </ul>
</li>
<li>It gets the IP address from the A record and sends the request.</li>
<li>When the request reaches our backend server's network, the firewall allows or blocks the request depending on whether HTTPS / HTTP is allowed.</li>
<li>The request reaches our server and goes to our reverse proxy, Nginx. The reverse proxy is used for many things, but one of them is to route to the right service. For example, if our server has both frontend and backend running, it routes to the backend.</li>
<li>The response is sent back to the user's browser.</li>
</ol>

<p><strong>Summary</strong></p>

<pre><code>user browser → DNS server → Network firewall → Server → Nginx → backend application
</code></pre>

</div>

<h3 id="c-why-do-we-need-a-backend-and-cant-we-do-everything-on-the-frontend">c. Why do we need a backend, and can't we do everything on the frontend?</h3>
<div style="margin-left: 2em;">

<p>Backend is usually used to work with data: the need to fetch data, receive data, persist data somewhere, and every kind of action that deals with data.</p>

<p>Why can't we do everything on the frontend, such that we would need a separate backend?</p>

<p>To understand why, we need to understand how frontends work.</p>

<pre><code>user browser → DNS server → Network firewall → Server → Nginx → frontend application
</code></pre>

<ol>
<li>When the user's browser makes a request to our frontend server, it receives the first document, which is an HTML document.</li>
<li>While the user's browser is parsing the HTML, it sees links to CSS, JS, images, etc., and makes individual requests for them.</li>
<li>The browser paints the page with the CSS content, and adds interactivity to the page from the JS content.</li>
</ol>

<p>A key thing to note is that for every user, their runtime is the browser. The browser executes the HTML, CSS, and JS code.</p>

<p>Why can't it also execute code to fetch, persist, and perform actions with data?</p>

<p>Browsers are built with security in mind because they execute code from the internet on the user's machine. Hence browsers are:</p>

<ol>
<li><strong>Sandboxed environments</strong> — isolated from the OS and filesystem, so the code can only access a limited amount of resources, e.g. DOM, Web APIs, external APIs. This prevents remote code from accessing files and data from the user's computer. A backend needs to access the underlying filesystem to write to a log file, and to access environment variables.</li>
<li><strong>CORS</strong> — limited access to external APIs via CORS.</li>
<li><strong>Database drivers</strong> — persisting data in a database requires a DB driver, and these are built for environments that handle socket connections, binary data, and maintain persistent connections. Browsers can't do that.</li>
<li><strong>Computing power</strong> — users access frontends from various devices. User devices may not have enough computing power to perform certain business logic.</li>
</ol>

</div>
</div>

---

<h2 id="3-http-protocol"><u>3. HTTP Protocol</u></h2>
<div style="margin-left: 2em;">

<p>Where it all begins.</p>

<p>When a user visits a website in their browser, the browser requests a webpage from a computer somewhere on the internet and displays it for the user. Computers on the internet are identified by IP address, hence the browser resolves the domain, e.g. http://myfrontend.com, to an IP address. The domain on the DNS server has an A record that points a domain name to an IP address.</p>

<p>That computer with the webpage is a server because it is a computer which clients, e.g. the user's browser or an application, request resources from.</p>

<p>Now that the IP address is found, there has to be a connection before the client and server can communicate.</p>

<p>There are various ways clients and servers communicate, and one of the most common is a protocol called HTTP, HyperText Transfer Protocol. It is a standardized way of communication between clients and servers. HTTP uses TCP as its underlying protocol for client server connection, because before any communication has to take place, a connection has to be established.</p>

<h3 id="i-two-ideas-at-the-heart-of-http">i. Two ideas at the heart of HTTP</h3>
<div style="margin-left: 2em;">

<p>HTTP has 2 main ideas at its heart.</p>

<ol>
<li>
<p><strong>Statelessness:</strong> When a client and server are communicating through HTTP request and response, servers do not keep any client session data, so they do not remember past requests. Hence the client has to send all the necessary information in the request to get the desired response.</p>put your stop loss at your 
<p>E.g. the client sends login credentials in a request and receives a response of successful login. Subsequent requests after that, if the client does not send the necessary information in the form of a cookie or token, the server will not remember that the client is logged in.</p>
</li>
<li>
<p><strong>Client server model:</strong> With HTTP protocol the client always initiates the communication. Servers do not initiate communications.</p>
</li>
</ol>

</div>

<h3 id="ii-http-versions">ii. HTTP Versions</h3>
<div style="margin-left: 2em;">

<p>Over the years HTTP evolved to improve client server interaction. Starting from:</p>

<ul>
<li><strong>HTTP/1.0:</strong> The TCP connection is closed after every request response cycle. Leading to latency and being resource intensive.</li>
<li><strong>HTTP/1.1:</strong> The same TCP connection can be used to send multiple requests and receive multiple responses.</li>
<li><strong>HTTP/2:</strong> Multiplexing. HTTP/1.1 had an issue called head of line blocking, where until the initial request has been responded to, all the current requests will have to wait. HTTP/2 sends many requests on the same TCP connection so one slow response does not block the others at the HTTP layer.</li>
<li><strong>HTTP/3:</strong> Uses QUIC over UDP as its underlying protocol for communication.</li>
</ul>

<p>HTTP/1.1 is still widely used today.</p>

</div>

<h3 id="iii-http-message">iii. HTTP message</h3>
<div style="margin-left: 2em;">

<p>A typical HTTP request has an HTTP method, a URL, HTTP version, HTTP headers, and an optional body.</p>

<p>A typical HTTP response has an HTTP version, a status code, HTTP headers, and an optional body.</p>

<p><strong>HTTP request example</strong></p>

<pre><code>PUT /api/users/12 HTTP/1.1
Host: backend.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json

{
    "name": "Ama Mensa"
}</code></pre>

<p><strong>HTTP response example</strong></p>

<pre><code>HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 50

{
    "msg": "User updated"
}</code></pre>

</div>

<h3 id="iv-http-headers">iv. HTTP Headers</h3>
<div style="margin-left: 2em;">

<p>HTTP headers are metadata which are key value pairs of an HTTP message. They provide certain information about the HTTP message like the content-type, date of the message, and security features.</p>

<h4 id="a-why-headers">a. Why headers</h4>
<div style="margin-left: 2em;">

<p>When sending a package, think of HTTP headers as the info that is pasted on the package, like where the package is going to, so others do not have to open the package to view its content.</p>

</div>

<h4 id="b-http-header-types">b. HTTP header types</h4>
<div style="margin-left: 2em;">

<ul>
<li><strong>Request headers</strong> — used to understand client env, preference and capabilities. E.g. User-Agent, Authorization, Accept, Cookie.</li>
<li><strong>General headers</strong> — general info about the HTTP message. E.g. Date, Cache-Control, Connection.</li>
<li><strong>Representation headers</strong> — info about the HTTP message's body so client and server know how to interpret them. E.g. Content-Length, Content-Encoding, ETag.</li>
<li><strong>Security headers</strong> — enhance the security of the HTTP request/response. E.g. Strict-Transport-Security (HSTS), Content-Security-Policy (CSP), Set-Cookie.</li>
</ul>

</div>

<h4 id="c-benefits-of-headers">c. Benefits of headers</h4>
<div style="margin-left: 2em;">

<p><strong>Extensible:</strong> HTTP headers make HTTP extensible because they give room to easily add custom headers for new features or security enhance without altering the underlying protocol.</p>

<p><strong>Remote control:</strong> Headers act as remote control, where clients can influence or control the kind of behaviour they want from the server. E.g. the client wants a response in Spanish instead of English with <code>Accept-Language: es</code>.</p>

</div>
</div>

<h3 id="v-http-methods">v. HTTP methods</h3>
<div style="margin-left: 2em;">

<p>HTTP methods are verbs which show the action the client wants the server to perform on a particular resource. HTTP messages show the client's intent on a particular resource.</p>

<p>There are a number of HTTP methods and the most used ones are as follows:</p>

<ul>
<li><strong>GET</strong> — get a resource from the server</li>
<li><strong>POST</strong> — create a resource on the server</li>
<li><strong>PUT</strong> — update and completely replace a resource on the server</li>
<li><strong>PATCH</strong> — update certain parts of a resource on the server</li>
<li><strong>DELETE</strong> — delete a resource from the server</li>
<li><strong>OPTIONS</strong> — used in CORS preflight request</li>
</ul>

<h4 id="a-idempotent-vs-non-idempotent">a. Idempotent vs Non Idempotent</h4>
<div style="margin-left: 2em;">

<p><strong>Idempotent</strong> means retry a request, or send the same request multiple times, will have the same effect on the resource. Idempotent methods: GET, PUT, DELETE.</p>

<p><strong>Non idempotent</strong> means retry a request, or sending a request multiple times, will have a different effect on the resource. Two POSTs can both return 201 and still create two rows. Non idempotent method: POST.</p>

</div>

<h4 id="b-cross-origin-resource-sharing-cors">b. Cross Origin Resource Sharing (CORS)</h4>
<div style="margin-left: 2em;">

<p>By default browsers have a same origin policy. For a simple cross origin request the browser still sends the request. It hides the response from JS if <code>Access-Control-Allow-Origin</code> does not match.</p>

<p>E.g. JS from the domain myfrontend.com makes a request for products from mybackend.com, which is a different domain. The browser noticing that adds an additional header to the request:</p>

<pre><code>Origin: https://myfrontend.com</code></pre>

<p>When the response comes from the server, the browser also checks the <code>Access-Control-Allow-Origin</code> header in the response. If the page's origin is not included, like <code>Access-Control-Allow-Origin: https://myfrontend.com</code> or <code>Access-Control-Allow-Origin: *</code>, the browser blocks the JS from getting access to the response.</p>

<p>There are two types of request that are made with CORS.</p>

<h5 id="i-simple-request">i. Simple request</h5>
<div style="margin-left: 2em;">

<p>Simple requests are requests made without a preflight OPTIONS request.</p>

</div>

<h5 id="ii-preflight-request">ii. Preflight request</h5>
<div style="margin-left: 2em;">

<p>Preflight is required to enquire the capabilities of the different domain's server.</p>

<p>There are a few things the browser checks to make a preflight request:</p>

<ol>
<li>The HTTP method is not GET, POST, HEAD. PUT and DELETE requests send a preflight request of OPTIONS before the actual request.</li>
<li><strong>OR</strong> the request includes non simple headers like Authorization or X-Custom-Header.</li>
<li><strong>OR</strong> the request has Content-Type other than application/x-www-form-urlencoded, multipart/form-data or text/plain.</li>
</ol>

<p>When an HTTP request to a different domain meets any of the criteria above, the browser sends a preflight request, which is an HTTP request with an OPTIONS method, to enquire the capabilities of the server.</p>

<p>E.g. a request made by the frontend to the backend to update a user's order:</p>

<pre><code>PATCH /api/order/1 HTTP/1.1
Host: mybackend.com
Origin: https://myfrontend.com
Content-Type: application/json
Authorization: Bearer eyuid...

{
    "status": "pending"
}</code></pre>

<p>will trigger a preflight request because the conditions were met.</p>

<pre><code>OPTIONS /api/order/1 HTTP/1.1
Host: mybackend.com
Origin: https://myfrontend.com
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: Authorization, Content-Type</code></pre>

<p>Server response:</p>

<pre><code>HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://myfrontend.com
Access-Control-Allow-Methods: PATCH, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 86400</code></pre>

<p>The server says it allows these methods on that URL, and this response will be the same for the next 86400 seconds, so do not make another preflight again. Then the original PATCH request is sent:</p>

<pre><code>HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://myfrontend.com
Content-Type: application/json

{
    "msg": "Order updated"
}</code></pre>

</div>
</div>
</div>

<h3 id="vi-http-response-code">vi. HTTP response code</h3>
<div style="margin-left: 2em;">

<p>Quickly communicate the result of a request in a standardized way. Standardized means consistency between how a server should communicate the results of the request. There should not be a case where the status code is telling a different message from the body. This standard is what our servers should follow.</p>

<ul>
<li><strong>1xx</strong> — information</li>
<li><strong>2xx</strong> — success</li>
<li><strong>3xx</strong> — redirect</li>
<li><strong>4xx</strong> — client error</li>
<li><strong>5xx</strong> — server error</li>
</ul>

<p><strong>1xx information</strong></p>
<p>Used for information to the clients. Can be used in a situation where the request is huge, when the headers are sent before the body.</p>
<ul>
<li><strong>100 Continue</strong> — headers received, continuing with the body.</li>
<li><strong>101 Switching Protocol</strong> — the communication protocol for the URL has changed, e.g. from HTTP to WebSocket.</li>
</ul>

<p><strong>2xx success</strong></p>
<ul>
<li><strong>200 OK</strong> — request was successful and the server sent requested resources or performed the requested action.</li>
<li><strong>201 Created</strong> — successful creating a new resource.</li>
<li><strong>204 No Content</strong> — successful but no content.</li>
</ul>

<p><strong>3xx redirection</strong></p>
<ul>
<li><strong>301 Moved Permanently</strong> — requested resource has been permanently moved to a new URL.</li>
<li><strong>302 Moved Temporarily</strong> — for now moved to a temporary URL.</li>
<li><strong>304 Not Modified</strong> — used in caching.</li>
</ul>

<p><strong>4xx client error</strong></p>
<ul>
<li><strong>400 Bad Request</strong> — client sends invalid data.</li>
<li><strong>401 Unauthorized</strong> — request requires valid authentication.</li>
<li><strong>403 Forbidden</strong> — refused to authorize the request.</li>
<li><strong>404 Not Found</strong> — resource that is not available.</li>
<li><strong>405 Method Not Allowed</strong> — this method is not allowed on this URL.</li>
<li><strong>409 Conflict</strong> — when resources are not unique, when a unique constraint is violated.</li>
<li><strong>429 Too Many Requests</strong> — rate limit client request.</li>
</ul>

<p><strong>5xx server error</strong></p>
<ul>
<li><strong>500 Internal Server Error</strong> — something broken or an exception was raised which was not handled.</li>
<li><strong>501 Not Implemented</strong> — this server does not implement that method. 405 is this method is not allowed on this URL.</li>
<li><strong>502 Bad Gateway</strong> — usually happened by proxies and load balancers, upstream invalid response.</li>
<li><strong>503 Service Unavailable</strong> — server is not available or too busy to handle more requests.</li>
<li><strong>504 Gateway Timeout</strong> — the upstream server failed to respond in time. Nginx did not receive a response from the original server.</li>
</ul>

</div>

<h3 id="vii-http-caching">vii. HTTP caching</h3>
<div style="margin-left: 2em;">

<p>Storing copies of responses for reuse. Reducing the need for repeated request to the server, reduces load time, bandwidth, server loads.</p>

<p>The client does not need to download a lot of data and the server does not need to send a lot if the data has not changed.</p>

<p>E.g. of a request response with caching:</p>

<pre><code>GET /api/products HTTP/1.1
Host: backend.com</code></pre>

<p>Response:</p>

<pre><code>HTTP/1.1 200 OK
Cache-Control: max-age=10, public
ETag: 3142
Last-Modified: Thu 17 Sep 2026 19:09:10 GMT

{
    "name": "product1"
}</code></pre>

<p><code>Cache-Control</code> is the header that tells the client to cache the response for a period of 10s as specified by <code>max-age=10</code>. While the cache is fresh, the client does not send another request. It uses the stored copy. <code>public</code> means the response can be stored in a shared cache.</p>

<p><code>ETag</code> is the hash value of the response which tells us that if the etag changed the response has also changed.</p>

<p><code>Last-Modified</code> as the name implies, the last time the response was modified.</p>

<p>After the cache is stale, the client may send the same request with:</p>

<pre><code>GET /api/products HTTP/1.1
Host: backend.com
If-None-Match: 3142
If-Modified-Since: Thu 17 Sep 2026 19:09:10 GMT</code></pre>

<p>The server responds that the response has not changed:</p>

<pre><code>HTTP/1.1 304 Not Modified
Cache-Control: max-age=10, public</code></pre>

<p>or it will respond with the new data, if the response has changed:</p>

<pre><code>HTTP/1.1 200 OK
Cache-Control: max-age=10, public
ETag: 2398
Last-Modified: Sat 19 Sep 2026 18:00:10 GMT

{
    "name": "product1 updated"
}</code></pre>

<p>Less data is sent when cached.</p>

</div>

<h3 id="viii-http-content-negotiation">viii. HTTP content negotiation</h3>
<div style="margin-left: 2em;">

<p>The mechanism client and server agree on the format to exchange data. Clients can indicate their preferred format and the server can respond with the format or a fallback format.</p>

<pre><code>GET /products HTTP/1.1
HOST: backend.com
Accept-Language: es
Accept: application/json
Accept-Encoding: gzip, deflate, br, zstd</code></pre>

<p>The server responds appropriately. In this case it responds in Spanish because of the header <code>Accept-Language: es</code>.</p>

</div>

<h3 id="ix-http-based-compression">ix. HTTP based compression</h3>
<div style="margin-left: 2em;">

<p>The client can tell the server which encodings it supports using <code>Accept-Encoding</code>, e.g. gzip, deflate, br, zstd.</p>

<p>The server can then compress the response body and set <code>Content-Encoding</code> so the client knows how to decode it. Compression reduces bandwidth. The client does not need to download a lot of data and the server does not need to send a lot.</p>

</div>

<h3 id="x-handling-large-request-and-response">x. Handling large request and response</h3>
<div style="margin-left: 2em;">

<p>A JSON body is small. Images, audio, video are not.</p>

<p><strong>Client to server. Multipart.</strong></p>

<p><code>multipart/form-data</code> is used to send files from the client to the server. The body is split into parts. That is why it is called multipart.</p>

<pre><code>POST /api/upload HTTP/1.1
Host: backend.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxk

------WebKitFormBoundary7MA4YWxk
Content-Disposition: form-data; name="file"; filename="shoe.jpg"
Content-Type: image/jpeg

&lt;binary image bytes&gt;
------WebKitFormBoundary7MA4YWxk--</code></pre>

<p><strong>Server to client. Chunked stream.</strong></p>

<pre><code>GET /videos/intro.mp4 HTTP/1.1
Host: backend.com
Accept: video/mp4

HTTP/1.1 200 OK
Content-Type: video/mp4
Transfer-Encoding: chunked
Connection: keep-alive

&lt;chunk of video bytes&gt;
&lt;chunk of video bytes&gt;
...</code></pre>

<p>Chunked means the file is sent in pieces. The connection stays open until the last chunk. Keep-alive means after the file, do not close the TCP connection.</p>

</div>
</div>

---

<h2 id="4-routing"><u>4. Routing</u></h2>
<div style="margin-left: 2em;">

<p>HTTP methods describe the intent, or what we want to do on that particular resource.</p>

<p>Routing defines where, or which resource we want to perform our intention on.</p>

<p>The server takes a combination of your intention and URL to map to a particular handler or some server side logic.</p>

<p>E.g. combination of intent, i.e. HTTP method GET, and URL <code>/users</code> maps to a handler/controller on the server side.</p>

<p>Routing is mapping a URL to server side logic.</p>

<pre><code>GET /api/books HTTP/1.1
POST /api/books HTTP/1.1</code></pre>

<p>The methods and route map to a particular handler on the server side to perform some logic.</p>

<h3 id="i-types-of-routing">i. Types of routing</h3>
<div style="margin-left: 2em;">

<h4 id="a-static-routes">a. Static routes</h4>
<div style="margin-left: 2em;">

<p><code>/api/books</code> — do not have a variable parameter inside the route.</p>

</div>

<h4 id="b-dynamic-route">b. Dynamic route</h4>
<div style="margin-left: 2em;">

<p><code>/api/books/:id</code> — have a variable inside the route. The dynamic part is called the path/route parameter.</p>

</div>

<h4 id="c-query-parameter">c. Query parameter</h4>
<div style="margin-left: 2em;">

<p>Different from path/URL parameter because it uses <code>?</code> and key-value to send some kind of filter in a GET request. E.g. <code>/api/search?query=:searchterm</code>.</p>

</div>

<h4 id="d-nested-route">d. Nested route</h4>
<div style="margin-left: 2em;">

<p><code>/api/user/:userid/books/:id</code></p>

</div>

<h4 id="e-route-versioning-and-deprecation">e. Route versioning and deprecation</h4>
<div style="margin-left: 2em;">

<p><code>/api/v1/products</code>, <code>/api/v2/products</code>.</p>

</div>

<h4 id="f-catch-all-route">f. Catch all route</h4>
<div style="margin-left: 2em;">

<p><code>/*</code> maps to a handler where the routing does not map to any handler.</p>

</div>
</div>
</div>

---

<h2 id="5-serialization-and-deserialization"><u>5. Serialization and Deserialization</u></h2>
<div style="margin-left: 2em;">

<p>Clients and servers communicate via a protocol, whether HTTP, gRPC or WebSocket. How can they make sense of that data being sent and received?</p>

<h3 id="i-the-language-barrier">i. The language barrier</h3>
<div style="margin-left: 2em;">

<p>There comes a language barrier problem. How does a client written in JS send data, usually a JS object, to a backend written in Java, and Java can make sense of that data or convert it to a Java class?</p>

<p>The fundamental issue: how can data created in one language be transmitted over a network, be accurately understood and reconstructed in a different programming language's data type?</p>

</div>

<h3 id="ii-solution">ii. Solution</h3>
<div style="margin-left: 2em;">

<p>To solve this cross language data exchange problem, client and servers agree on a common format to send data that they both understand.</p>

<p>Serialization and deserialization is converting data to and from a common format so that during transmission or storage, it is language agnostic. So clients and servers from different languages are able to make sense of that data.</p>

</div>

<h3 id="iii-the-flow">iii. The flow</h3>
<div style="margin-left: 2em;">

<pre><code>client application → serialize to agreed upon format → network infrastructure transmits over the network
server application → receives the transmitted serialized format → deserialized into language data structure
server does some business logic
server application → serialize from language data structure to agreed upon format → network transmits serialized format</code></pre>

</div>

<h3 id="iv-osi-model">iv. OSI model</h3>
<div style="margin-left: 2em;">

<p>Standard on how computers and network devices communicate with each other.</p>

<p>OSI has 7 layers: Physical Layer, Data link layer, Network layer, Transport layer, Session layer, Presentation Layer, Application layer.</p>

<p>It defines how data are converted in different formats at different layers and transmitted over a network.</p>

<p>As a backend engineer my responsibility starts and ends at the application layer. I define how my application data structures and objects are serialized for transmission, and how data received is deserialized in my application's native format.</p>

<p>Conversion to TCP segments, IP packets, Ethernet frames and physical signals are handled by the OS and network infrastructure.</p>

</div>

<h3 id="v-serialization-standards">v. Serialization standards</h3>
<div style="margin-left: 2em;">

<ul>
<li><strong>Text based serialization format:</strong> JSON, YAML, XML</li>
<li><strong>Binary serialization format:</strong> Protocol Buffers, Avro, etc.</li>
</ul>

<p>JSON is the most popular serialization standard during client and server communication.</p>

</div>
</div>


<!-- 

























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


HTTP protocol

It is a standardized way of communication between clients and servers. HTTP uses TCP as its underlying protocol for client server connection because before any communication has to take place, a connection has to be established.


Core ideas in HTTP 

HTTP has 2 main ideas at its heart. 

1. Statelessness: When a client and server are communicating, through HTTP request and response, servers do not any client session data, so do not remember past requests, hence the client has to send all the necessary information in the request to get the desired response. 

eg Client sends login credentials in a request and receives a response of successful login. 
Subsequent requests after that, if the client does not send the necessary information in the form of a cookie or token, the server will not remember that the client is logged in.


2. Client server model: With HTTP Protocol the client always initiates the communication. Servers do not initiate communications


HTTP versions
Over the years HTTP evolved to improve client server interaction
Starting from 

HTTP/1.0: The TCP connection is closed after every request response cycle. Leading to latency and being resource intensive. 

HTTP/1.1: The same TCP connection can be used to send multiple requests and receive multiple responses. 

HTTP/2: Multiplexing. HTTP/1.1 had an issue called head of line blocking where until the initial request has been responded to, all the current requests will have to wait. HTTP/2 sends many requests on the same TCP connection so one slow response does not block the others at the HTTP layer.

HTTP/3: uses QUIC over UDP as its underlying protocol for communication. 

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
HTTP methods are verbs which show the action the client wants the server to perform on a particular resource.

Http messages show clients intent on a particular resource.

there are number of http method and the most used ones are as follows 

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


HTTP response codes 
Quickly communicate the result of a request in a standardized way. Standardized means consistency between how a server should communicate the results of the request. There should not be a case where the status code is telling a different messsage from the body. 

This standard is what our servers should follow. 

1xx - information
2xx - success
3xx - redirect
4xx - client error
5xx - server error 

1xx information 
used for information to the clients. 
Can be used in a situation where the request is huge when the headers are sent before the body so 

100 Continue - in a situation where the request is huge and the headers are sent before the body so 100 indicates headers received, contining with the body

101 Switching Protocol - Where the communication protocol for the url has changed eg from HTTP to websocket. 


2xx - success response
200 OK - request was successful and server sent requested resources or performing the requested action

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
409 CONFLICT - when resources are not unique. when a unique constraint is violated
429 TOO MANY REQUEST - rate limit client request


5xx - Server error 
500 Internal Server Error - something broken or exception was raised which was not handled.
501 Not implemented - this server does not implement that method. 405 is this method is not allowed on this URL.
502 Bad gateway - usually happened by proxies and load balancers, upstream invalid response
503 Service Unavailable - Server is not available or too busy to handle more requests.
504 gateway timeout - the upstream server failed to respond in time, nginx did not receive a response from original server



HTTP headers 
Are metadata which are key value pairs of an HTTP message. they provide certain information about the HTTP message like the content-type, date of the message 
security features. 

why headers 
When sending a package think of HTTP headers as the info that is pasted on the package like where the package is going to etc so other do not have to open the package to view it's content.

types of headers 
request headers - used to understand client env, preference and capabilities- eg User-Agent, Authorization, Accept, Cookie etc

general header - general info about the HTTP message - eg Date, Cache-Control, Connection etc.

representation header - info about the HTTP message's body ensure client, server knows how to interpret them - eg content-length, content-encoding, etag

security headers - enhance the security of the http request/response- eg Strict-Transport-Security(HSTS), Content-Security-Policy(CSP), Set-Cookie


Benefits of HTTP header

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
public means the response can be stored in a shared cache 



ETag is the hash value of the response which tells us that if the etag changed the response has also changed

Last-Modified as the name implies, the last time the response was modified


After the cache is stale, the client may send the same request with

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

                    attacker can send thousands of request and because their is not rate limit, 
                    all the request will go through

                    2 things can happen
                        1. They will find a match and access to user account through brute forcing.
                        2. overload server until it crashs

                    implement layers of rate limit 
                        1. per ip 
                        2. per failed attempt on an account
                        3. global rate limit as a number of failed login in attempts in a given timeframe



            3. Authorization
                user are in our system. what are they permitted to do or see?

                false sense of security after user is authenticated. 

                issues 
                    Broken object level authorization: 
                        Our system does not check authorization of user on individual objects at the database layer but check authorization 
                        in the routing layer.    

                        fix 
                            eventhough we check authorization in our routing layer, read invoices, 



                        user being able to pass auth checks and authorization checks like read:invoices 
                        does not mean they have access to all invoices in our system

                    Broken function role authorization 
                        instead of checking granular permission like read:voices 
                        makes normal user access to an endpoint that reads all the invoices like /admin/invoices
                        we need an extra middleware should create the role of the user at the role level 
                        so only admins are authorized to use a specific route. 



                    indirect object reference

                    broken access control patterns 

                prevent authorization attacks 
                    centralized place of auth logic
                    default deny 
                    automated testing of authorization specfically 
                    audit logs every time a sensitive resource is accesssed, it should be logged , authorization check fails 



            3. Cross site scription XSS
                attacker manage to get any js to execute in a users browser in the content of our website/

user provide content we have to be careful
    storing it
    processing it 
    file handling

we have to be every careful of how we are handling it 
what kind of access we have given, resource 

xss user provided markup like html, markup with a hidden script tag that is stored 
and how once we can get out of 99% of vulnerabilities 



Content security policies 
    http header sent by server to browser

routing 
types of routes




4. Routing 

    Http methods describes the intent or what we want to do on that particular resource

    Routing define where or wwhich resource we want to perform our intention on. 

    The server takes a combination of your intention and url to map to a particular handler or some server side logic
    eg 

    Combination of intent ie HTTP method eg GET, and url /users maps to a handler/controller on the server side

    routing is mapping a url parameter to a server side logic 



    GET /api/books HTTP/1.1
    POST /api/books HTTP/1.1 
    the methods and route mapps to a particular handler on the server side to perform some logic 


    types of routing 
        1. Static routes: /api/books do not have a variable parameter inside the route

        2. Dynamic route /api/books/:id have a variable inside the route. The dynamic part is called the path/route parameter

        3. Query parameter different from path/url parameter because it uses ? and key-value 
        to send some kind of filter in a get request.  /api/search?query=:searchterm 

        4. Nested route /api/user/:userid/books/:id

        5. route versioning and depreciation /api/v1/products, /api/v2/products. 

        6. catch all route: /* maps handler where the routing do not map to any handler.



5. Serialization and Deserialization




Clients and server commnunicates via a protocol, whether http, grpc or websocket. 
How can they make sense of that data being sent and recieved. 

There comes a language barrier problem. How does a client written in js sends data usually a js object to a bakend write in java
and java can make sense of that data or convert it to a java class. 

foundamental issue  how can data created in one language be transmitted over a network, be accurately 
understood and reconstructed in a different programming languages data type. 

solution 
    To solve this cross language data exchange problem. Client and servers agrees on a common format to send data
    that they both understand.

Serialization and deserialization is converting data to and from a common format so that during transmission or storage, 
it is language agnostic. so client and servers from different languages are able to make sense of that data. 


Hence the flow 
    client application -> serialize to agreed upon format -> network infrastructure transmits over the network
    server application -> recieves the transmitted serialized format -> deserialized into language data structure, 
    server does some business logic 
    server application -> serialize from language data structure  to agreed upon format -> network transmitted serialized format


OSI model 
Standard on how computer and network devices communicate with each other. 

OSI has 7 layers

Physical Layer, Data link layer, Network layer, Transport layer, Session layer, Presentation Layer, Application layer
It defines how data are converted in different format at different layers and transmitted over a network.
As a backend engineer my responsible starts and end at the application layer. 
I define how my application data strucuture and objects are serialized for transmission and how data recieved at deserialized 
in my application native format. 

Conversion to TCP segements, IP packets, Ethernet frames and physical signals are handled by the OS and network infrastructure


serialization standards 

text based serialization format, json, yaml, xml 
binary serialization format : protocol buffers, avro etc

JSON is most popular serialization standard during client and server communication









How does a client being a js application for example in the browser and a server a java app makes 
sense of the http request and response

eg js app sends a js object in the request body. how does the server make sense of the request 
body in its own data type. 

osi model 
    high level understanding 
    top level application layer
    bottom physical layer. 

problem is two machine in different 
sent data to another machine in a way which is language agnostic

common standard client should send data in a particular standard / format and also be able convert from 
that format

server should also be able to convert from that standard
common standard that both client and server agrees to send and recieve data.



alot of serialization standards.
most popular is json, yaml, xml. text based serialization
binary format protocol, avro etc 



reference article 

serialization and deserialization
https://markstack-app.vercel.app/blogs/serialization-and-deserialization-data-flow-between-clients-and-servers
https://medium.com/@ali.gelenler/serialization-deserialization-mechanisms-7cc37b6d948b








Controller, service, repository 
    Request lifecycle in ths server


    1. Request reaches the server. OS forward the http request to the port our backend is listen on
    2. Server is listening, server has routing algorithm, depending on the algo, we match the request to a particular handler/controller
    3. Server application has handler , services and repository why is there a seperation, why cant we just have one big fun that accept request, do some processing and query and return a response. 

    4. it is a design pattern to our code based is scalable, easy to add features and debug issues. 

    5. in each handler, we recieve two objects request and response objects
    6. deserialize the json object into your lang native data format, like java , request failed send client 400 Bad request
    7. production grade there should be a validation layer. we should validate every input to our server, validate request body, query params, path params etc

    client data is in the expected format. optionally transformation, some modification for our server to process the request

    controller layer calls the service layer with all the data it has transform
    service layer do not dealing with http related stuff    






































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













Rest API Design

    Tim berners lee - invented www in 1990
    not accounted scalability
    Roy Fielding, introduced 6 constraints to solve scalability
    together http/1.1
    Fielding names constraints as REST architectural standards in 2000
    Idempotency
        POST non idempotent method
    if we have a non crud operation or an operation where u can't determine the method to use
    POST    custom actions any thing that is not a crud operation 
    rest api specification.

    reason for following standards
        - limit confusion and assumptions 
            eg delete method for fetching resource using ur services, less bugs and confusions

    Things to do when design apis
        1. Start from ui design, to have an idea how user are going interact with data
        2. The resources, nouns from the ui client requirements etc
        3. Database schema
        4. API interface design

    list api 
        pagination, filtering, sorting


    





    Tim Berners Lee started a project called www to share information with the whole world
    within a year he invented uri, http, html, web sever, web browser, wysiwg. 

    Within a short there was expotential growth and he did not account for this scale.
    To accomodate this large user based new technique and standards has to be introduced 

    Roy Fielding was consence about the web scaling problems, to address the issue he proposed a
    couple of constraints 
    1. Client Server - seperate of concern between client and server. client handles user experience, while server
        handles data storage and business logic. 
        allow each component to evolve indepenedently improve scala

    2. Uniform interface
        a standard way of web components to communicate with each other. 
        consistent interface across all services 

    3. Layered system 
        architecture composed of hierarchical layer and each layer can only see and interact with 
        the layer below it. scalable , security, ability to add intermediate components , like load balancers 
        and proxy servers.

    4. Cache 
        Server should label resources as cacheable or not by the client. leads reduce server load, improve network efficency
        , improve user 

    5. stateless - each request from the client must contain all the necessary information to process the request

    6. Code on demand (optional) - server extends clients function by transfering code to the client like js




Designing APIs 
    different ways of design apis graphql, grpc etc
    Rest api is the common standard of designing apis

    Developers have many questions when writing, if they are stick to the standards , what methods to use for custom actions
    that are not crud, should they use plural or singular for resource names, 
    worrying about whether their api is restful or not. 

    A little history 

    1990 Tim Berners Lee started a project called the worldwide web (www) to share knowledge with the whole
        within are invented URI, HTTP Protocol, HTML, webserver, html editor in browser, wysiwyg

        the project was ending breakdown because of the expotential growth of user, it did not account for scale.

    1993 Roy Fieldings introduced to some 6 constraints help with scale of the web

        1. Client server - model seperate of concerns clients handles user experience, server handles data storage and business logic
                allowing each component to scale independently

        2. Uniform interface - standardized way of different components communicating with each other. 

        3. Layered system - the architecture consists of hierarchical layers and each layer interact with the layer below
                allows scalability and security and ability to add intermediate layers like 
                load balancers, proxies etc. 

        4. Cacheable - responses from server should be explicitly labeled as cacheable or not. Client will the cache the reponse to reduce server loads , improve network efficency, enhance user experience by providing faster response

        5. Statelessness - client should add all the necessary information in the request to a server for the server to process all the necessary information. server does not store any client context between require. 

        improve scalability since any server can handle client requests.

        6. Code on demand it is optinal, server can extend clients functionality by transferring executable codes to client.

    What does REST means
        The name REST stands for Representational State Transfer
            Web resources can be represented in different formats depending on the client or server
            eg. Your profile information stored in a db table as a column can be representated in json, html , xml 
            depending on the client and server

            State current condition of the resource that is the actual data stored in a db. The current information
            ur profile stored in db. 

            when a request is made for the profile, the server does not send raw object but sends a representation 
            of the state in a specific format like json, xml

            transfer movement of that state in a representational format between clients and servers
            using a http protocol.


        Rest is an architectural style where one resource is particular condition can be transferred in different formats using an
        underlying protocol http between clients and servers. 

        eg. server store my current profile info in a database, and sends it to a client in a different format like json, html
        xml instead of the db table object using the underlying http protocol

        clients can send the update info of my profile original in js object as a different format like json to the server using http

        and the follow the 6 specific constraints to make the system more scalable. 

    Idempotency 
        perform same actions multiple times has the same effect as performing it once. 
        does not matter how many times you retry the same request, it will have the same effect on the resource

        get, post, patch, put, delete others head, options

        POST 
            is non idempotent 
            open end rest api makes the post method open end, meaning whenever we have an action that is not 
            a crud , we can put it under post 


Why follow standards like rest api standards 

    eliminate confusion and assumption in our workflow 
        if ur api use delete for post operations it leaves more for error and confusion


Things to do when developing api

    1. Start from ui design, to have an idea how user are going interact with your data
    2. Identify resources, nouns from the ui client requirements etc
    3. Database schema
    4. API interface design

        design the interface first, not the business logic
        resources are lowercase plural in the path
        list and create share the same url, method is what differs
            GET /organizations   list   200
            POST /organizations  create 201, return the new entity


        list apis supports
            pagination  query params page and limit
            sorting     sortBy and sortOrder
            filtering   query params named after fields e.g. status=archived

        get / update / delete a single resource share the same path, method differs
            GET    /organizations/:id  200
            PATCH  /organizations/:id  200, return updated entity
            DELETE /organizations/:id  204 empty body
            prefer PATCH over PUT, we usually send partial fields not a full replace

        404 vs empty list
            404 only when client asks for a specific id that does not exist
            list with no matches is still 200 with empty array, never 404

        custom actions are not crud
            archive org, clone project etc
            even if it updates a field, extra server work (delete related data, emails) makes it a custom action
            POST /organizations/:id/archive
            POST /projects/:id/clone
            status code follows what actually happened, not the method
            archive can be 200, clone can be 201 if a new resource was created

        same patterns for every resource (org, project, task)
        json fields camelCase
        keep names consistent across resources, description not dsc
        same route style everywhere, always plural

        after design
            1. interactive docs like swagger
            2. keep routes, payloads, query params consistent and intuitive
            3. provide sane defaults (page, limit, sort, status=active on create)
            4. no abbreviations in field names










url structure
    https://michael.com/blogs

    https - scheme
    michael.com - domain
    api.michael.com - supdomain
    /blogs - resources

  


FULL TRANSCRIPT

    API design is something you will spend a lot of time on as a backend engineer.
    The problem is people still get confused: should the path be plural or singular,
    should update be PATCH or PUT, which method for a custom action, which status code.
    The standards already exist. The purpose is to extract rules from them and stick to
    one consistent styling so we can focus on business logic instead of worrying whether
    the API is RESTful.

    History
        Around 1990 Tim Berners-Lee started the World Wide Web to share knowledge.
        He invented URI, HTTP, HTML, the first web server, the first browser, and the first WYSIWYG editor.
        The web grew faster than it was designed for, so it needed new techniques to scale.

        Roy Fielding proposed constraints that later became REST:
            client-server     UI on the client, data and business logic on the server, they evolve independently
            uniform interface a standardized way for components to communicate
            layered system    each layer only talks to the layer below it, so you can add proxies and load balancers
            cacheable         the server must label responses as cacheable or not
            stateless         each request contains all the information the server needs; the server does not remember the previous request
            code on demand    optional; the server can send executable code like JavaScript to the client

        Together they wrote HTTP 1.1. In 2000 Fielding named this architectural style REST: Representational State Transfer.

    What REST means
        Representational    resources are represented in a format: JSON, XML, HTML, depending on the client
        State               the current attributes of a resource, for example a cart's items, quantities, and total
        Transfer            those representations move between client and server over HTTP

    URL
        scheme (https) + domain / subdomain (api.example.com) + version (/v1) + resource path + query params
        In the path the resource should always be plural. Even when fetching one book it is /books/:id, not /book/:id.
        Do not put spaces or underscores in URLs. For slugs: make it lowercase and replace spaces with hyphens.
        A forward slash means a hierarchical relationship: a collection, then one item from that collection.

    Idempotency
        Performing the same action multiple times has the same effect as performing it once.
        The side effect on the server stays the same. We are talking about the side effect you cause, not someone else changing data in between.

        GET is idempotent. It is a fetch. You do not change server state.
        PATCH and PUT are idempotent. Sending the same payload again leaves the resource in the same state.
        DELETE is idempotent. The first call deletes it. Later calls may return 404, but they cause no further side effect.
        POST is non-idempotent. Each call can create a new resource with a new id.

        PATCH updates part of a resource. PUT replaces the whole representation.
        Internally people mix them. For a public API stick to the semantic standard.
        In single page applications we usually send partial JSON fields, so PATCH is the usual update.

        POST is also open-ended. If an action is not create, read, update, or delete, put it under POST.
        Send email is a custom action. It is not a fetch, not a create, not an update, not a delete. Use POST.

    Why follow standards
        If you do not follow standards, consumers have to read your code or try methods until something works.
        That is a lot of room for error and assumption.
        If you follow the standard, most behavior is already known. Integration is faster. Fewer bugs. Less guesswork.

    How to start designing the interface
        The first thing, before you write any business logic, is designing the interface.
        It should be intuitive, delightful to use, not vague, and it should follow REST standards.
        We are designing the interface point, not the execution point. No programming language, no database code yet.

        Start from the UI design, Figma or wireframes, so you see how the end user interacts with data.
        Resources are nouns you can identify from the requirements: projects, users, organizations, tasks, tags.
        Then database schema, then API interface design.
        List the actions from CRUD: fetch all, get one, create, update, delete. Then design those in Insomnia.

    CRUD
        List and create share the same URL. The server differentiates by method. GET lists. POST creates.
        Path is lowercase plural: /organizations
        Usually APIs have versioning, /v1, but this demo skips it.
        Client payload excludes server-handled fields: id, createdAt, updatedAt.
        POST create returns 201 and the newly created entity.
        GET list returns 200.

        Pagination
            Do not return every row. Return a portion of the data.
            Serializing a thousand records is heavy. The user only sees 10 or 20 at a time.
            Response has data, total, page, totalPages.
            Query params: limit and page.
            If the client sends nothing, the server should set defaults: page 1, limit 10 or 20.

        Sorting
            sortBy and sortOrder.
            If you do not sort, each call can return a random order because the database does not store rows in sequence.
            Default sort is createdAt descending, the latest entries. If sortOrder is omitted, default descending.

        Filtering
            Query params named after fields, for example status=archived.

        Get one, update, and delete share the same route: /organizations/:id. Only the method differs.
        Update is PATCH. Returns 200 and the updated entity.
        GET one returns 200.
        DELETE returns 204 No Content, empty body. Successful, but there is no content to send.

        404 is when the client requests a particular resource id that does not exist.
        A list with no data is still 200 with an empty array. Never send 404 on a list call.

    Custom actions
        Archiving is not just PATCHing the status field.
        The server may delete related projects and tasks, send emails, etc.
        That is a custom action: POST /organizations/:id/archive
        Do not assume every POST returns 201. Custom actions that did not create a resource return 200.

        Clone is a custom action too: POST /projects/:id/clone
        You cannot fake clone with create, because the server may also clone tasks and send email.
        If clone creates a new resource, it can return 201.
        Status code follows what happened on the server, not the method name.

        JSON fields should always follow camelCase, in payloads and in responses.

    Consistency
        Across resources, query parameters and JSON payloads should look similar.
        If create organization uses description, create project should also use description, not dsc.
        Clients assume the next API works like the first one they integrated. Do not make them guess.
        Same for routes. If organizations is plural, projects should be plural, not project.
        Stick to the style. Do not make arbitrary decisions across different resources.

    Closing
        Provide interactive documentation, Swagger / OpenAPI, from the start. Use it to test and as docs.
        Make APIs intuitive and consistent. Follow REST. If you cannot, follow something and stick to it.
        Provide sane defaults: page, limit, sort, and on create only require what you absolutely need.
        If status is omitted, default to active. That is a safe assumption.
        Avoid abbreviations. You have more context than the people integrating the API. Keep fields readable.

        A REST API is designed first. It is not coded right away.
        Dedicate a separate session to designing the interface without thinking about programming language.
