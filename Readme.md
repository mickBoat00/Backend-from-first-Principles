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






