# my_notes
This Repo contain my notes on various topics !!
WebSockets
HHTP Requests : Whenever we hit google.com, request goes to web server and response is send. There will be TCP connection establish and response is send, once the response is recieved the connection is closed. Everytime a request is made as new connection is established. HTTP - UniDirectional.

WebSockets: There will be client and server. Websockets are bi-direction communication. Connection is open till any of the client or server closes the connection. Websockets are used for real time application, gaming applications and chat appliactions. eg:/wss://ws-feed.com Websockets should not be used whenever we dont require real time updates. IF we want to load the data only once, then HTTP or restful webservices.

Ajax uses HTTP connection.Everything it used new socket and poll the data.
