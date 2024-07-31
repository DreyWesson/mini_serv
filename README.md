Overview
This implementation describes a basic TCP server designed to manage multiple client connections simultaneously. It leverages the select function for I/O multiplexing, allowing the server to efficiently handle multiple connections without using multithreading.

Data Structure
To ensure proper encapsulation and avoid using global variables, the server uses a linked-list data structure to manage client connections. This choice not only simplifies debugging and maintenance but also enhances the extensibility of the server's functionalities.

Initialization
The server is initialized with a port number provided as a command-line argument. It's important to ensure that the port number is within the valid range (0 < n <= 65535).

The initServer function sets up the server's data structure, initializing various fields and preparing the file descriptor sets used by select. Key functions used in this process include bzero (or memset), which initializes our structure, and FD_ZERO, which initializes the file descriptor sets (activefds, writefds, and readfds).

While bzero and FD_ZERO have overlapping functionalities, they serve distinct purposes. FD_ZERO is a macro specifically designed to initialize a file descriptor set to ensure it is properly prepared for use with select and related functions.

Socket Creation
The createSock function is responsible for creating the server socket using the socket function. It then adds the socket descriptor to the activefds set and updates max_fd to keep track of the highest file descriptor currently in use.

Address Configuration and Binding
The server’s address, including the IP and port, is configured in the configAddr function. The bindAndListen function then binds the server to this address and sets it up to listen for incoming connections using the listen function.

Main Loop for Handling Connections
The server enters an infinite loop in the handleCon function, continuously waiting for activity on any of the file descriptors—whether new connections or data from existing connections.

Monitoring File Descriptors
The select function is used to monitor multiple file descriptors, blocking until activity is detected. Before each call to select, the readfds and writefds sets are updated with the current active file descriptors.

Handling Different Types of Activity
New Connections: When a new connection is detected on the server socket, the acceptRegistration function is called. This function accepts the new connection, registers the client's file descriptor in the activefds set, adds the client to the linked list, and notifies all other clients of the new connection.

Client Messages: When a client sends data, the processMessage function handles the incoming message. The data is read from the socket, appended to any existing message data, and processed to extract complete messages (delimited by a newline character).

Message Handling
The server uses the extract_message function to separate complete messages from partial ones. Once a complete message is extracted, it is sent to all other clients using the sendNotification function.

Client Management
Registration: When a new client connects, registerClient adds them to the server's list of clients and notifies the other clients of the new connection.

Deregistration: If a client disconnects or an error occurs, deregisterClient removes them from the list, cleans up resources, and notifies the other clients of the disconnection.

Error Handling
The fatalError function handles critical errors by cleaning up resources and terminating the program. This includes freeing memory for all clients and closing sockets.

Memory Management and Cleanup
Memory management is carefully handled to prevent leaks. When clients are removed or messages are no longer needed, the corresponding memory is freed. Functions like freeClient and deleteAll ensure proper cleanup of resources.

Conclusion
This TCP server implementation efficiently manages multiple client connections using the select function for I/O multiplexing. It continues to run, handling new connections and messages from connected clients, until it is manually stopped or encounters an unrecoverable error.
