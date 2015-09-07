# sslengine.implementation

Implementation of a Java SSL/TLS server and client, making use of JSSE framework and specifically the SSLEngine class.

## An introduction to JSSE

JSSE is the standard way Java provides to implement SSL/TLS communication. One of its core classes is `SSLContext`, which you can easilly configure and equally easily get an input and output stream from. These streams though will be blocking, since `available()` will always return false for SSL/TLS connections. In order to achive a non-blocking SSL/TLS solution, JSSE provides the `SSLEngine`, which leads to a more complicated solution, since the developer has to implement parts of the protocol himself and also decide the way the transport link will be implemented. Due to the lack of examples i was able to find in the internet, i decided to start a project, in order to explore JSSE, and share it here. More information about JSSE can be found in this link: https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html

## The implementaton

There are 3 Java classes in the project:

* `NioSslPeer` is an abstract class that represents a generic SSL/TLS peer, which is not specified yet neither as a client nor as a server. It implements the handshake protocol, which is common for clients and servers, and involves the exchange of configuration messages, in order the connection to be established. As already mentioned, the implementation of the transport link is made by the developer and in this project Java nio SocketChannel was chosen.
* `NioSslServer` extends `NioSslPeer` to run as a server. In order to start a server you have to instantiate an object with the IP address and port it will listen to and the protocol it will apply. 

        NioSslServer server = new NioSslServer("TLSv1.2", "localhost", 9222);
        server.start();
        
  After this, the server will wait for new connections and serve the requests that arrive to it. Server is configured to serve the SSL/TLS requests in a very trivial way: it just sends back a "Hello i am your server" response. A Java nio Selector is used in order to serve all clients within a single Thread. To gracefully shutdown the server you can call `server.stop()`.
* NioSslClient extends `NioSslPeer` to acts as an SSL/TLS client. To use a client you have to instantiate an object and connect to a running server.

        NioSslClient client = new NioSslClient("TLSv1.2", "localhost", 9222);
        client.connect(); 

  After the successful connection to the server, client provides `write(String message)` and `read()` methods, that send a string message to its peer and wait to get feedback from it respectively. Read method is blocking. In order to gracefully shutdown a client you can use `client.shutdown()`.

## Example
In project's test directory there is a Demo class that contains an example of `NioSslClient`'s and `NioSslServer`'s usage. It creates and starts a thread running a server and then creates 4 clients, connect them to the server, send messages to it and read the responses.

## Get the project
You can clone the project and build it with maven to run the example and check the code. I used Maven 3.3, Java 1.8 and log4j 1.2 for the logging. The project is under MIT license, so I would be glad if you would like to contribute!
