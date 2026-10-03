# Java TCP Chat System

A multi-client, client-server chat application built in Java using TCP socket programming. The system demonstrates network communication, concurrent client handling, thread-based message processing, and shared client state management.

## Overview

This project implements a simple real-time chat system using a **client-server architecture**.

A central Java server listens for TCP connections from multiple clients. Each connected client is handled independently using a dedicated thread, allowing multiple users to communicate concurrently. Messages received by the server are broadcast to the connected clients.

The project was built to explore practical concepts in **Java networking, socket programming, concurrency, exception handling, and client-server communication**.

## Features

* TCP-based client-server communication
* Multiple simultaneous client connections
* Unique nicknames for connected users
* Real-time message broadcasting
* Dedicated server thread for each connected client
* Separate client thread for receiving incoming messages
* Join and leave notifications
* Graceful client disconnection using `bye`
* Concurrent management of connected clients
* Basic error handling for network and I/O failures

## Architecture

The application follows a simple client-server architecture:

```text
                    ┌─────────────────┐
                    │   Java Server   │
                    │   TCP : 1234    │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Client 1 │   │ Client 2 │   │ Client 3 │
        └──────────┘   └──────────┘   └──────────┘
```

Each client establishes a TCP connection to the server.

The server creates a dedicated `ClientHandler` thread for each connection. Connected clients are tracked using a thread-safe `ConcurrentHashMap`, allowing the server to broadcast messages to active users.

## How It Works

### 1. Server Startup

The server creates a `ServerSocket` and listens on TCP port `1234`.

```java
ServerSocket serverSocket = new ServerSocket(PORT);
```

It continuously waits for incoming client connections.

### 2. Client Connection

A client connects to the server using the server address and port:

```java
Socket socket = new Socket(serverAddress, PORT);
```

The client then provides a nickname to identify itself in the chat.

### 3. Concurrent Client Handling

For every new connection, the server creates a dedicated `ClientHandler` and starts it in a separate thread.

```java
ClientHandler clientHandler = new ClientHandler(socket);
new Thread(clientHandler).start();
```

This allows multiple clients to communicate with the server concurrently.

### 4. Message Broadcasting

When a client sends a message, the server receives it through the client's input stream and broadcasts it to all currently connected clients.

The server maintains connected clients using:

```java
ConcurrentMap<PrintWriter, String> clientWriters
```

The concurrent collection helps safely manage shared client state while multiple client-handler threads are active.

### 5. Client-Side Message Handling

The client uses a separate thread to listen for incoming messages.

This allows the user to continue entering messages while messages from other users are received asynchronously.

```text
Main Thread
    │
    ├── Read user input
    └── Send messages

Message Listener Thread
    │
    └── Receive and display messages
```

### 6. Disconnecting

A client can leave the chat by sending:

```text
bye
```

The server removes the client's connection from the active client collection and notifies the remaining users that the client has left.

## Project Structure

```text
chatsystem-distributed/
│
├── Client.java
├── Server.java
└── README.md
```

### `Server.java`

Responsible for:

* Starting the TCP server
* Accepting client connections
* Creating client-handler threads
* Managing connected clients
* Receiving client messages
* Broadcasting messages
* Handling client disconnections

### `Client.java`

Responsible for:

* Connecting to the server
* Sending the user's nickname
* Sending messages
* Receiving messages
* Displaying incoming messages
* Closing the connection when the user exits

## Requirements

* Java Development Kit (JDK) 8 or later
* Terminal or command prompt
* Network access between the client and server if running on different machines

Check your Java installation:

```bash
java -version
javac -version
```

## Running the Application

### 1. Clone the repository

```bash
git clone https://github.com/Nonga001/chatsystem-distributed.git
cd chatsystem-distributed
```

### 2. Compile the source files

```bash
javac Server.java Client.java
```

This generates the required `.class` files locally.

### 3. Start the server

```bash
java Server
```

The server will listen on:

```text
Port: 1234
```

You should see:

```text
Server is listening on port 1234...
```

### 4. Start a client

Open another terminal:

```bash
java Client localhost
```

Enter a nickname when prompted.

### 5. Start additional clients

Open additional terminals and run:

```bash
java Client localhost
```

Each client can then communicate through the server.

## Example

Client 1:

```text
Enter your nickname:
Alice

You: Hello everyone!
```

Client 2 receives:

```text
Alice has joined the chat.
Alice: Hello everyone!
```

Another client can respond:

```text
You: Hi Alice!
```

The server broadcasts the message to the connected clients.

## Technical Concepts Demonstrated

This project demonstrates several fundamental software-engineering and networking concepts:

### Java Networking

Uses Java's networking APIs including:

* `ServerSocket`
* `Socket`
* `InputStream`
* `OutputStream`
* `BufferedReader`
* `PrintWriter`

### TCP Communication

The application uses TCP to establish reliable communication between clients and the server.

### Concurrency

The server creates a separate thread for each connected client, while the client uses a separate listener thread for incoming messages.

### Thread-Safe Shared State

Connected clients are stored using `ConcurrentHashMap`, allowing concurrent client handlers to access and update shared state.

### Exception Handling

Network and I/O operations are wrapped with exception handling to prevent failures from terminating the application unexpectedly.

### Resource Management

Sockets and streams are managed using Java's try-with-resources mechanism where appropriate, helping ensure resources are closed correctly.

## Design Considerations

### Why TCP?

TCP provides reliable, ordered, connection-oriented communication, which makes it suitable for a basic chat application where messages should arrive reliably.

### Why Multiple Threads?

A blocking socket operation can wait indefinitely for network input. Dedicated threads allow the server to continue accepting and servicing other clients while one client is waiting for input.

### Why `ConcurrentHashMap`?

The server can have multiple `ClientHandler` threads accessing the connected-client collection simultaneously. A concurrent collection helps avoid unsafe concurrent modifications to shared state.

## Current Limitations

This is an educational networking project and intentionally keeps the architecture simple.

Current limitations include:

* No user authentication
* No message persistence
* No encryption
* No database integration
* No message history
* No advanced room management
* Server state is lost when the server stops
* No automatic client reconnection
* No graphical user interface

## Possible Improvements

Future versions could introduce:

* User authentication and authorization
* TLS encryption for network communication
* Persistent message storage using a relational database
* Chat rooms and private messaging
* Client reconnection handling
* Message timestamps
* Structured message formats such as JSON
* Logging and monitoring
* Automated unit and integration tests
* Executor-based thread management
* A graphical or web-based client

## Learning Outcomes

Through this project, I gained practical experience with:

* Java socket programming
* TCP/IP client-server communication
* Multithreaded programming
* Concurrent data structures
* Network I/O
* Exception handling
* Resource management
* Git-based software development
* Debugging distributed client-server behaviour

## Author

**Shaldon Omondi Nonga**

BSc Computer Science
Dedan Kimathi University of Technology

GitHub: https://github.com/Nonga001
