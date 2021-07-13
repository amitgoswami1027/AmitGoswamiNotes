# SYSTEM DESIGN BASICS 

## Software Architecture Tiers - What is Tier??
Think of a tier as a logical separation of components in an application or a service. And when I say separation, I mean physical separation at the component level, not the code level. 
What do I mean by components?
* Database
* Backend application server
* User interface
* Messaging
* Caching

![image](https://user-images.githubusercontent.com/13011167/115166957-4d476400-a0d3-11eb-8968-0560c758ff75.png)

### Single Tier Applications: A single-tier application is an application where the user interface, backend business logic & the database all reside in the same machine.
Typical examples of single-tier applications are desktop applications like MS Office, PC Games or an image editing software like Gimp.

* Advantage: The main upside of single-tier applications is they have no network latency since every component is located on the same machine. This adds up to the performance of the software. In single-tier apps, the data is easily & quickly available since it is located in the same machine.Though it largely depends on how powerful the machine is & the hardware requirements of the software, to gauge the real performance of a single-tier app. The data of the user stays in his machine & doesn’t need to be transmitted over a network. This ensures data safety at the highest level.
* Disadvantage: One big downside of single-tier app is that the business has no control over the application. The code in single-tier applications is also vulnerable to being tweaked & reversed engineered. The security, for the business, is minimal. The applications’ performance & the look and feel can get inconsistent as it largely depends on the configuration of the user’s machine.

### Two Tier Applications: A Two-tier application involves a client and a server. The client would contain the user interface & the business logic in one machine. And the backend server would be the database running on a different machine. The database server is hosted by the business & has control over it.

* Need of Two Tier Application : Good example of this is the online browser & app-based games or  to-do list app or a similar planner or a productivity app. The game files are pretty heavy, they get downloaded on the client just once when the user uses the application for the first time. Moreover, they make the network calls only to keep the game state persistent. Also, fewer server calls mean less money to be spent on the servers which is naturally economical.

### Three Tier Applications: In a three-tier application, the user interface, application logic & the database all lie on different machines & thus have different tiers. They are physically separated. So, if we take the example of a simple blog, the user interface would be written using Html, JavaScript, CSS, the backend application logic would run on a server like Apache & the database would be MySQL. A three-tier architecture works best for simple use cases.

![image](https://user-images.githubusercontent.com/13011167/115167314-cabfa400-a0d4-11eb-87ab-dd02bba55d72.png)

### N Tier Applications: An N-tier application is an application which has more than three components involved. 
What are those components?
* Cache
* Message queues for asynchronous behaviour
* Load balancers
* Search servers for searching through massive amounts of data
* Components involved in processing massive amounts of data
* Components running heterogeneous tech commonly known as web services etc.

* Why The Need For So Many Tiers: Two software design principles that are key to explaining this are the Single Responsibility Principle & the Separation of Concerns.
Single Responsibility Principle simply means giving one, just one responsibility to a component & letting it execute it with perfection. Be it saving data, running the application logic or ensuring the delivery of the messages throughout the system.This approach gives us a lot of flexibility & makes management easier.
Separation of concerns kind of means the same thing, be concerned about your work only & stop worrying about the rest of the stuff. These principles act at all the levels of the service, be it at the tier level or the code level.

### Difference Between Layers & Tiers - Don’t confuse tiers with the layers of the application. Some prefer to use them interchangeably. But in the industry layers of an application typically means the user interface layer, business layer, service layer, or the data access layer. The layers mentioned in the illustration are at the code level. The difference between layers and tiers is that the layers represent the organization of the code and breaking it into components. Whereas, tiers involve physical separation of components. All these layers together can be used in any tiered application. Be it single, two, three or N-tier.

#### Note: There is another name for n-tier apps, “distributed applications.” But, I don’t think it’s safe to use the word “distributed” yet, as the term distributed brings along a lot of complex stuff with it. At this point, it would confuse rather than help us. Although I will discuss distributed architecture in this course, for now, we will just stick with the term n-tier applications.

# DISTRIBUTED SYSTEMS
Key characteristics of a distributed system include Scalability, Reliability, Availability, Efficiency, and Manageability.
#### SCALABILITY -  Scalability is the capability of a system, process, or a network to grow and manage increased demand. A system may have to scale because of many reasons like increased data volume or increased amount of work, e.g., number of transactions. A scalable system would like to achieve this scaling without performance loss. 
#### RELIABILITY - Reliability is the probability a system will fail in a given period. A distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.
#### AVAILABILITY - Availability is the time a system remains operational to perform its required function in a specific period.It is a simple measure of the percentage of time that a system, service, or a machine remains operational under normal conditions. 
* Reliability Vs. Availability - If a system is reliable, it is available. However, if it is available, it is not necessarily reliable. In other words, high reliability contributes to high availability, but it is possible to achieve a high availability even with an unreliable product by minimizing repair time and ensuring that spares are always available when they are needed.
#### EFFICIENCY - To understand how to measure the efficiency of a distributed system, let’s assume we have an operation that runs in a distributed manner and delivers a set of items as result. Two standard measures of its efficiency are the response time (or latency) that denotes the delay to obtain the first item and the throughput (or bandwidth) which denotes the number of items delivered in a given time unit (e.g., a second). The two measures correspond to the following unit costs:
* Number of messages globally sent by the nodes of the system regardless of the message size.
* Size of messages representing the volume of data exchanges.
However, it is quite difficult to develop a precise cost model that would accurately take into account all these performance factors; therefore, we have to live with rough but robust estimates of the system behavior.
#### SERVICEABILITY & MAINTAINABILITY - Serviceability or manageability is the simplicity and speed with which a system can be repaired or maintained; if the time to fix a failed system increases, then availability will decrease. 

# SYSTEM DESIGN PATTERNS
## 1. BLOOM FILTERS
PROBLEM: If we have a large set of structured data (identified by record IDs) stored in a set of data files, what is the most efficient way to know which file might contain our required data? We don’t want to read each file, as that would be slow, and we have to read a lot of data from the disk. One solution can be to build an index on each data file and store it in a separate index file. This index can map each record ID to its offset in the data file. Each index file will be sorted on the record ID. Now, if we want to search an ID in this index, the best we can do is a Binary Search. Can we do better than that?

The Bloom filter data structure tells whether an element may be in a set, or definitely is not. The only possible errors are false positives, i.e., a search for a nonexistent element might give an incorrect answer. With more elements in the filter, the error rate increases. An empty Bloom filter is a bit-array of m bits, all set to 0. There are also k different hash functions, each of which maps a set element to one of the m bit positions.

A Bloom filter is a data structure designed to tell you, rapidly and memory-efficiently, whether an element is present in a set. The price paid for this efficiency is that a Bloom filter is a probabilistic data structure: it tells us that the element either definitely is not in the set or may be in the set he base data structure of a Bloom filter is a Bit Vector. 

Suppose you are creating an account on Geekbook, you want to enter a cool username, you entered it and got a message, “Username is already taken”. You added your birth date along username, still no luck. Now you have added your university roll number also, still got “Username is already taken”. It’s really frustrating, isn’t it? But have you ever thought how quickly Geekbook check availability of username by searching millions of username registered with it. There are many ways to do this job –  
* Linear search : Bad idea!
* Binary Search : Store all username alphabetically and compare entered username with middle one in list, If it matched, then username is taken otherwise figure out , whether entered username will come before or after middle one and if it will come after, neglect all the usernames before middle one(inclusive). Now search after middle one and repeat this process until you got a match or search end with no match.This technique is better and promising but still it requires multiple steps. But, There must be something better!!

Bloom Filter is a data structure that can do this job. A Bloom filter is a space-efficient probabilistic data structure that is used to test whether an element is a member of a set. For example, checking availability of username is set membership problem, where the set is the list of all registered username. The price we pay for efficiency is that it is probabilistic in nature that means, there might be some False Positive results. False positive means, it might tell that given username is already taken but actually it’s not.

![image](https://user-images.githubusercontent.com/13011167/120114529-2f265680-c19d-11eb-996a-29e269f9b043.png)

The element X definitely is not in the set, since it hashes to a bit position containing 0.
For a fixed error rate, adding a new element and testing for membership are both constant time operations, and a filter with room for ‘n’ elements requires O(n)O(n)O(n) space.

## 2. Consistent Hashing
The act of distributing data across a set of nodes is called data partitioning. A naive approach will use a suitable hash function that maps the data key to a number. Then, find the server by applying modulo on this number and the total number of servers.
Use the Consistent Hashing algorithm to distribute data across nodes. Consistent Hashing maps data to physical nodes and ensures that only a small set of keys move when servers are added or removed

## 3. Quorum
In Distributed Systems, data is replicated across multiple servers for fault tolerance and high availability. Once a system decides to maintain multiple copies of data, another problem arises: how to make sure that all replicas are consistent, i.e., if they all have the latest copy of the data and that all clients see the same view of the data?
In a distributed environment, a quorum is the minimum number of servers on which a distributed operation needs to be performed successfully before declaring the operation’s overall success.


## WEB ARCHITECTURE
Web architecture involves multiple components like database, message queue, cache, user interface & all running in conjunction with each other to form an online service.
![image](https://user-images.githubusercontent.com/13011167/115167818-73bace80-a0d6-11eb-9bc2-bd738a4bcc54.png)

Client-Server architecture is the fundamental building block of the web. The architecture works on a request-response model. The client sends the request to the server for information & the server responds with it. The client holds our user interface. The user interface is the presentation part of the application. It’s written in Html, JavaScript, CSS and is responsible for the look & feel of the application.

## Client : There are primarily two types of clients
* Thin Client : Thin Client is the client which holds just the user interface of the application. It has no business logic of any sort. For every action, the client sends a request to the backend server. Just like in a three-tier application.

![image](https://user-images.githubusercontent.com/13011167/118360858-a7d6c180-b5a6-11eb-89d7-b331082dadb4.png)

* Thick Client: On the contrary, the thick client holds all or some part of the business logic. These are the two-tier applications. The typical examples of Fat clients are utility apps, online games etc.

![image](https://user-images.githubusercontent.com/13011167/118360982-e7051280-b5a6-11eb-839c-4005725180e1.png)

## Server : The primary task of a web server is to receive the requests from the client & provide the response after executing the business logic based on the request parameters received from the client. Every service, running online, needs a server to run. Servers running web applications are commonly known as the application servers. 
All the components of a web application need a server to run. Be it a database, a message queue, a cache or any other component. In modern application development, even the user interface is hosted separately on a dedicated server.

## Server Side Rendering : Often the developers use a server to render the user interface on the backend & then send the rendered data to the client. The technique is known as server-side rendering. I will discuss the pros & cons of client-side vs server-side rendering further down the course.


## Communication Between the Client & the Server 
The client & the server have a request-response model. The client sends the request & the server responds with the data.
 * HTTP Protocol : The entire communication happens over the HTTP protocol. It is the protocol for data exchange over the World Wide Web. HTTP protocol is a request-response protocol that defines how information is transmitted across the web. It’s a stateless protocol, every process over HTTP is executed independently & has no knowledge of previous processes.
* REST API & API Endpoints : The backend application code has a REST-API implemented which acts as an interface to the outside world requests. Every request be it from the client written by the business or the third-party developers which consume our data have to hit the REST-endpoints to fetch the data.

### WHAT IS REST? REST stands for Representational State Transfer. It’s a software architectural style for implementing web services. Web services implemented using the REST architectural style are known as the RESTful Web services.
* With the implementation of a REST API the client gets the backend endpoints to communicate with. This entirely decouples the backend & the client code.
* REST Endpoint: An API/REST/Backend endpoint means the url of a service. For example, https://myservice.com/users/{username} is a backend endpoint for fetching the user details of a particular user from the service.The REST-based service will expose this url to all its clients to fetch the user details using the above stated url.
* Decoupling Clients & the Backend Service : With the availability of the endpoints, the backend service does not have to worry about the client implementation. Developers can have different implementations with separate codebases, for different clients, be it a mobile browser, a desktop browser, a tablet or an API testing tool. Introducing new types of clients or modifying the client code has no effect on the functionality of the backend service.

### Application Development Before the REST API 
* Before the REST-based API interfaces got mainstream in the industry. We often tightly coupled the backend code with the client. JSP (Java Server Pages) is one example of it.
* We would always put business logic in the JSP tags. Which made code refactoring & adding new features really difficult as the logic got spread across different layers. Also, in the same codebase, we had to write separate code/classes for handling requests from different types of clients. A different servlet for a mobile client and a different one for a web-based client
* After the REST APIs became widely used, there was no need to worry about the type of the client. Just provide the endpoints & the response would contain data generally in the JSON or any other standard data transport format. And the client would handle the data in whatever way they would want.
* This cut down a lot of unnecessary work for us. Also, adding new clients became a lot easier. We could introduce multiple types of new clients without considering the backend implementation.
* In today’s industry landscape, there is hardly any online service without a REST API. Want to access public data of any social network? Use their REST API.

#### The REST-API acts as a gateway, as a single entry point into the system. It encapsulates the business logic. Handles all the client requests, taking care of the authorization, authentication, sanitizing the input data & other necessary tasks before providing access to the application resources.

![image](https://user-images.githubusercontent.com/13011167/118361617-e0c46580-b5a9-11eb-83e2-4a7225345da6.png)

## HTTP PUSH & PULL
### HTTP PULL: As I stated earlier, for every response, there has to be a request first. The client sends the request & the server responds with the data. This is the default mode of HTTP communication, called the HTTP PULL mechanism.
* The client pulls the data from the server whenever it requires. And it keeps doing it over and over to fetch the updated data.An important thing to note here is that every request to the server and the response to it consumes bandwidth. Every hit on the server costs the business money & adds more load on the server.
* What if there is no updated data available on the server, every time the client sends a request?
* The client doesn’t know that, so naturally, it would keep sending the requests to the server over and over. This is not ideal & a waste of resources. Excessive pulls by the clients have the potential to bring down the server.

### HTTP PUSH: In this mechanism, the client sends the request for particular information to the server, just for the first time, & after that the server keeps pushing the new updates to the client whenever they are available.
* The client doesn’t have to worry about sending requests to the server, for data, every now & then. This saves a lot of network bandwidth & cuts down the load on the server by notches.
* This is also known as a Callback. Client phones the server for information. The server responds, Hey!! I don’t have the information right now but I’ll call you back whenever it is available.

### HTTP PULL - Polling with Ajax ( Asynchronous Javascript & XML) : There are two ways of pulling/fetching data from the server :
* The first is sending an HTTP GET request to the server manually by triggering an event, like by clicking a button or any other element on the web page.
* The other is fetching data dynamically at regular intervals by using AJAX without any human intervention.

* AJAX – Asynchronous JavaScript & XML : AJAX stands for asynchronous JavaScript & XML. The name says it all, it is used for adding asynchronous behaviour to the web page

![image](https://user-images.githubusercontent.com/13011167/118362060-81675500-b5ab-11eb-9382-4afa4d01748d.png)

#### Steps:
* As we can see in the illustration above, instead of requesting the data manually every time with the click of a button. AJAX enables us to fetch the updated data from the server by automatically sending the requests over and over at stipulated intervals.
* Upon receiving the updates, a particular section of the web page is updated dynamically by the callback method. We see this behaviour all the time on news & sports websites, where the updated event information is dynamically displayed on the page without the need to reload it.
* AJAX uses an XMLHttpRequest object for sending the requests to the server which is built-in the browser and uses JavaScript to update the HTML DOM.
* AJAX is commonly used with the Jquery framework to implement the asynchronous behaviour on the UI.
* This dynamic technique of requesting information from the server after regular intervals is known as Polling.

### HTTP PUSH : 
* TTL : In the regular client-server communication, which is HTTP PULL, there is a Time to Live (TTL) for every request. It could be 30 secs to 60 secs, varies from browser to browser.If the client doesn’t receive a response from the server within the TTL, the browser kills the connection & the client has to re-send the request hoping it would receive the data from the server before the TTL ends this time.
* Open connections consume resources & there is a limit to the number of open connections a server can handle at one point in time. If the connections don’t close & new ones are being introduced, over time, the server will run out of memory. Hence, the TTL is used in client-server communication.
* But what if we are certain that the response will take more time than the TTL set by the browser? ---> PERSISTENT CONNECTIONS ARE REQUIRED????

#### Persistent Connection : A persistent connection is a network connection between the client & the server that remains open for further requests & the responses, as opposed to being closed after a single communication. It facilitates HTTP Push-based communication between the client and the server.

![image](https://user-images.githubusercontent.com/13011167/118362307-aa3c1a00-b5ac-11eb-8807-77850b19faf3.png)

#### Heartbeat Interceptors: Now you might be wondering how is a persistent connection possible if the browser kills the open connections to the server every X seconds? The connection between the client and the server stays open with the help of Heartbeat Interceptors. These are just blank request responses between the client and the server to prevent the browser from killing the connection.
* Isn’t this resource-intensive?
* Yes, it is. Persistent connections consume a lot of resources in comparison to the HTTP Pull behaviour. But there are use cases where establishing a persistent connection is vital to the feature of an application.
* For instance, a browser-based multiplayer game has a pretty large amount of request-response activity within a certain time in comparison to a regular web application.
* It would be apt to establish a persistent connection between the client and the server from a user experience standpoint.
* Long opened connections can be implemented by multiple techniques such as Ajax Long Polling, Web Sockets, Server-Sent Events etc.

## HTTP Push-Based Technologies
### Web Sockets : A Web Socket connection is ideally preferred when we need a persistent bi-directional low latency data flow from the client to server & back.

Typical use-cases of these are messaging, chat applications, real-time social streams & browser-based massive multiplayer games which have quite a number of read writes in comparison to a regular web app.

With Web Sockets, we can keep the client-server connection open as long as we want.

Have bi-directional data? Go ahead with Web Sockets. One more thing, Web Sockets tech doesn’t work over HTTP. It runs over TCP. The server & the client should both support web sockets or else it won’t work.

### AJAX – Long Polling : Long Polling lies somewhere between Ajax & Web Sockets. In this technique instead of immediately returning the response, the server holds the response until it finds an update to be sent to the client.

The connection in long polling stays open a bit longer in comparison to polling. The server doesn’t return an empty response. If the connection breaks, the client has to re-establish the connection to the server.

The upside of using this technique is that there are quite a smaller number of requests sent from the client to the server, in comparison to the regular polling mechanism. This reduces a lot of network bandwidth consumption.

Long polling can be used in simple asynchronous data fetch use cases when you do not want to poll the server every now & then

### HTML5 Event Source API & Server Sent Events: The Server-Sent Events implementation takes a bit of a different approach. Instead of the client polling for data, the server automatically pushes the data to the client whenever the updates are available. The incoming messages from the server are treated as Events.

Via this approach, the servers can initiate data transmission towards the client once the client has established the connection with an initial request.

This helps in getting rid of a huge number of blank request-response cycles cutting down the bandwidth consumption by notches.

To implement server-sent events, the backend language should support the technology & on the UI HTML5 Event source API is used to receive the data in-coming from the backend.

An important thing to note here is that once the client establishes a connection with the server, the data flow is in one direction only, that is from the server to the client.

SSE is ideal for scenarios such as a real-time feed like that of Twitter, displaying stock quotes on the UI, real-time notifications etc

### Streaming Over HTTP : Streaming Over HTTP is ideal for cases where we need to stream large data over HTTP by breaking it into smaller chunks. This is possible with HTML5 & a JavaScript Stream API.

The technique is primarily used for streaming multimedia content, like large images, videos etc, over HTTP.

Due to this, we can watch a partially downloaded video as it continues to download, by playing the downloaded chunks on the client.

To stream data, both the client & the server agree to conform to some streaming settings. This helps them figure when the stream begins & ends over an HTTP request-response model

## Client-Side Rendering : When a user requests a web page from the server & the browser receives the response. It has to render the response on the window in the form of an HTML page. For this, the browser has several components, such as the:
* Browser engine
* Rendering engine
* JavaScript interpreter
* Networking & the UI backend
* Data storage etc.
The rendering engine constructs the DOM tree, renders & paints the construction. And naturally, all this activity needs a bit of time.

## Server-Side Rendering : To avoid all this rendering time on the client, developers often render the UI on the server, generate HTML there & directly send the HTML page to the UI. This technique is known as the Server-side rendering. It ensures faster rendering of the UI, averting the UI loading time in the browser window since the page is already created & the browser doesn’t have to do much assembling & rendering work.

The server-side rendering approach is perfect for delivering static content, such as WordPress blogs. It’s also good for SEO as the crawlers can easily read the generated content.

However, the modern websites are highly dependent on Ajax. In such websites, content for a particular module or a section of a page has to be fetched & rendered on the fly.

Therefore, server-side rendering doesn’t help much. For every Ajax-request, instead of sending just the required content to the client, the approach generates the entire page on the server. This process consumes unnecessary bandwidth & also fails to provide a smooth user experience.

A big downside to this is once the number of concurrent users on the website rises, it puts an unnecessary load on the server.

Client-side rendering works best for modern dynamic Ajax-based websites.

Though we can leverage a hybrid approach, to get the most out of both techniques. We can use server-side rendering for the home page & for the other static content on our website & use client-side rendering for the dynamic pages.




## System Design Readings

* [MultiTenant Design] https://success.outsystems.com/Support/Enterprise_Customers/Maintenance_and_Operations/Designing_the_Architecture_of_Your_OutSystems_Applications/Designing_Scalable_Multi-Tenant_Applications
* [MultiTenant Design] https://docs.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns
* [MultiTenant Design] https://github.com/uglide/azure-content/blob/master/articles/sql-database/sql-database-design-patterns-multi-tenancy-saas-applications.md
* [MultiTenant Design] https://engineering.opsgenie.com/how-to-design-a-modern-multi-tenant-saas-application-with-auth0-45c446e825b7
* [HTTP Protocol] : https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview
* [Server Sent Events]: https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
