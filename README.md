# Java TCP Chat System

A clean, lightweight multi-client chat application built in Java using TCP socket programming. The system demonstrates fundamental networking concepts, concurrent client handling, thread-based message processing, and shared client state management.

## Overview

This project implements a real-time chat system using a **client-server architecture**.

A central Java server listens for TCP socket connections from multiple clients. Each connected client is handled independently using a dedicated worker thread, enabling multiple users to communicate concurrently. Messages received by the server are broadcast in real time to all connected clients.

The project demonstrates practical concepts in **Java networking, socket programming, concurrency, input/output stream management, and client-server communication**.

## Features

* **TCP Client-Server Communication**: Reliable connection-oriented networking using Java Sockets.
* **Concurrent Client Handling**: Dedicated server thread for each connected client connection.
* **Unique Nicknames**: Prompt-based client identification upon connection.
* **Real-Time Message Broadcasting**: Distributes messages from any client to all active participants.
* **Asynchronous Client Listener**: Client uses a separate thread to receive and display incoming messages without blocking user input.
* **Connection Lifecycle Notifications**: Automatic broadcasting of user join and leave notifications.
* **Graceful Disconnection**: Client connection teardown triggered by typing `bye`.
* **Thread-Safe Client Tracking**: Uses `ConcurrentHashMap` to safely manage active client writers across threads.

## Architecture

The system follows a standard multithreaded client-server architecture:

```text
Client 1 ─┐
Client 2 ─┼──> Java TCP Server (Port 1234) ──> Connected Clients
Client 3 ─┘
```

### Flow Breakdown

```text
┌──────────┐              ┌──────────────┐              ┌──────────────────┐
│  Client  │ ──(Connect)─>│ ServerSocket │ ──(Accept)──>│  ClientHandler   │
└──────────┘              └──────────────┘              │     (Thread)     │
     │                                                  └────────┬─────────┘
     │                                                           │
     ├────── Send Nickname ─────────────────────────────────────>│ (Registers Client)
     │                                                           │
     ├────── Send Message ──────────────────────────────────────>│ (Broadcasts to all)
     │                                                           │
     └────── Send 'bye' ────────────────────────────────────────>│ (Cleans up & closes)
```

1. **Server Initialization**: `Server` creates a `ServerSocket` bound to TCP port `1234` and listens for incoming connections in a loop.
2. **Connection Acceptance**: When a client connects, the server accepts the socket and spawns a `ClientHandler` runnable on a new `Thread`.
3. **Registration & Join Broadcast**: The `ClientHandler` prompts the client for a nickname, adds the client's `PrintWriter` to a shared `ConcurrentHashMap`, and broadcasts a join notification.
4. **Message Loop & Broadcasting**: The handler reads incoming lines from the client. Each message is printed to the server console and broadcast to all active `PrintWriter` streams.
5. **Asynchronous Client Reading**: On the client side, a dedicated `messageListener` thread continuously reads from the server input stream while the main thread handles user console input.
6. **Graceful Exit**: When a client sends `bye` (or closes the stream), the server broadcasts a leave notification, removes the client from `ConcurrentHashMap`, and closes the socket.

## Project Structure

```text
chatsystem-distributed/
├── src/
│   ├── Client.java     # TCP Client implementation with asynchronous message listener
│   └── Server.java     # TCP Server & concurrent ClientHandler implementation
├── .gitignore          # Excludes compiled binaries, IDE configs, logs, and OS files
└── README.md           # Project documentation
```

### `src/Server.java`

* **`Server`**: Starts the `ServerSocket` on port `1234` and accepts client connections in an infinite loop.
* **`ClientHandler`**: A `Runnable` class executed in a separate thread per client connection. Manages stream initialization, nickname registration, message reading, broadcasting across all connected clients via `ConcurrentHashMap`, and socket cleanup.

### `src/Client.java`

* **`Client`**: Connects to the server on port `1234`. Prompts the user for a nickname, launches a background thread to listen for incoming server messages, and processes user console input to send messages until `bye` is entered.

## Technologies Used

* **Language**: Java (JDK 8+)
* **Networking Protocol**: TCP/IP
* **Networking API**: Java Sockets (`java.net.ServerSocket`, `java.net.Socket`)
* **Concurrency**: Java Threads (`java.lang.Thread`), Thread-safe collections (`java.util.concurrent.ConcurrentHashMap`)
* **I/O Streams**: Java Standard I/O (`java.io.BufferedReader`, `java.io.PrintWriter`, `java.io.InputStreamReader`)
* **Version Control**: Git / GitHub

## How to Run

### Prerequisites

* Java Development Kit (JDK 8 or later) installed and available on your system path.

Verify Java installation:

```bash
java -version
javac -version
```

### 1. Clone the Repository

```bash
git clone https://github.com/Nonga001/chatsystem-distributed.git
cd chatsystem-distributed
```

### 2. Compile the Project

Compile the Java source files from `src/` into an `out/` directory:

```bash
javac -d out src/*.java
```

### 3. Start the Server

Run the compiled `Server` class from the `out/` directory:

```bash
java -cp out Server
```

Expected server output:

```text
Server is listening on port 1234...
```

### 4. Start a Client

Open a new terminal window and run:

```bash
java -cp out Client localhost
```

*(Replace `localhost` with the server's IP address if connecting over a local network.)*

### 5. Start Additional Clients

Open additional terminal windows and launch more clients:

```bash
java -cp out Client localhost
```

Each client will prompt for a nickname and join the shared chat session. Type `bye` to exit.

## Technical Concepts Demonstrated

* **TCP Socket Programming**: Establishing reliable, connection-oriented communication between client and server endpoints.
* **Client-Server Architecture**: Centralized server coordinating communication and state among distributed clients.
* **Multithreading**: Spawning independent execution threads on both server (per-client handler) and client (message listener) to prevent blocking I/O operations.
* **Thread Safety & Concurrency Control**: Utilizing `ConcurrentHashMap` to allow safe, thread-safe access and mutation of active client connection references across concurrent threads.
* **I/O Stream Handling**: Wrapping byte streams with character streams (`BufferedReader`, `PrintWriter`) for efficient line-based text processing.
* **Resource Management**: Utilizing try-with-resources and explicit `finally` cleanup blocks to properly release sockets and streams upon disconnection.
* **Exception Handling**: Handling `IOException` gracefully to ensure socket teardown without server crashes.

## Limitations

This repository is designed as a foundational networking and concurrency project. It currently has the following scope limitations:

* **No Authentication**: Clients are identified solely by self-reported nicknames without password verification.
* **No Encryption / Security**: Transmission is unencrypted plain text over standard TCP sockets (no TLS/SSL).
* **No Persistence**: Chat history is transient and not stored in a database or file system.
* **No Private Messaging / Rooms**: All messages are broadcast to every connected client globally.
* **No Command Protocol**: Lacks structured command routing (e.g., JSON or binary framing).
* **Thread-per-Client Scaling Limit**: Uses basic thread creation per client rather than a thread pool (`ExecutorService`) or non-blocking I/O (`java.nio`).

## Future Improvements

* **TLS/SSL Encryption**: Secure socket communication using `SSLSocket` and `SSLServerSocket`.
* **User Authentication**: Implement registration and login with secure password hashing.
* **Message Persistence**: Store messages in a database (e.g., PostgreSQL or SQLite) for message history retrieval.
* **Private Messaging & Chat Rooms**: Add support for targeted user messaging and topic-based channels.
* **Thread Pooling / NIO**: Use `ExecutorService` or Java NIO (`Selectors`) for improved scalability under high connection loads.
* **Graphical User Interface**: Develop a desktop GUI (JavaFX / Swing) or web interface.
* **Structured Protocol**: Adopt JSON framing or Protocol Buffers for message payload serialization.

## Author

**Shaldon Omondi Nonga**  
BSc Computer Science  
Dedan Kimathi University of Technology  

GitHub: [https://github.com/Nonga001](https://github.com/Nonga001)
