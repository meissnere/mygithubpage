# Erich Project 1 README File

Available at https://erichmeissner.com/data/OSreadme.md

## Purpose

The purpose of the project is to design a multithreaded client-server file transfer. The web-server serves static files and the client server acts as a load generator.

This readme file serves as a continuous thought process for my project. My understanding of multithreading, how I approach each part of the project, and what resources I'll use to complete it will be included in this documentation.

## Step 1: Warm Up Sockets

Since we are building a client-server file transfer, how will the client and server communicate? They will communicate via sockets. Luckily, my knowledge from the Computer Networks GT class helped to familiarize me with the Python standard socket API. I hope I find these exercises in the C socket API simple.

## Step 2: Transferring a File (Second Part of Warmup)
Calls to `send()` are not guaranteed. The Operating System does not guarantee that `send()` will copy the contents of the full range of memory that is specified to the socket. This uncertainty is due to the network interface being a shared resource. The caller will be notified of how much the OS actually copied via a return value. Therefore, the OS can copy as much of the memory range as it can fit in the network buffer.

```c
// Do not assume that sent has the same value as length
// The function send() may need to be called again (with different
// parameters) to ensure that the full message is sent
assert( sent == length);
```
Now, for the `recv()` function, something similar is the case. The operating system may return the call prematurely; the OS does not wait for the range of memory specified to be completely filled from the network. This circumstance occurs for TCP sockets. The memory address in a `recv()` function may only receive a portion of the originally sent data; therefore, the caller will know how much data the OS wrote via a return value.

This behavior might be desirable; if the user does not want to store all of the data in memory, then data received over the network is save somply to disk. For this warmup, all we do is set up a client that saves received data to a file on disk. Read and write access (`S_IRUSR | S_IWUSR`) should be taken into consideration for file permissions issues.

Try sending a chunk of data at a time to save on memory. The server should not stop after a single transfer, rather should continue accepting and transferring files to other clients:
```c
while(true) {
	recv();
	send();
}
```

## Part 1: Implemengting a Getfile Protocol
Successful transfer of a file from one computer to another is illustrated below:
![Getfile Transfer Figure](illustrations/gftransfer.png)

Remember, Getfile is a protocol made up for this project. The general form of a request is

```
<scheme> <method> <path>\r\n\r\n
```

Note:

* The scheme is always GETFILE.
* The method is always GET.
* The path must always start with a ‘/’.

The general form of a response is

```
<scheme> <status> <length>\r\n\r\n<content>
```

My attempt at Getfile is outline in this picture:
![Getfile Attempt](cs6200.png)

## Part 2: Implementing a Multithreaded Getfile Server
A bodd-worker thread pattern is desired for both the client and the server. This pattern will be graded. Futhermore, `fork()` is not allowed to spawn child processes. Instead, the `<pthread.h>` library should be employed to create and manage threads.

This section will have us implement our own version of the handler since the part 1 Getfile server can only handle a single request at a time. The `gfserver_main.c` will have to be uploaded as needed as changes to the `handler.c` file are made.

The stucture of the program is as follows:

* The main (boss) thread will continue listening to the socket and accepting new connection requests.
* Worker threads will full handle any new connections, however. The number, or pool, of these worker threads will be initialized to the number of threads specified as a command line argument.

On the client side, similar functionality will be implemented. The client will become multithreaded by modifying `gfclient_download.c`. The Getfile workload can only download a single file; when server latencies are large, the clients are at a major disadvantage. For this reason, we are making the client mutlithreaded. Similarly, the client pool of worker threads will be initialized based on the number of client worker threads specified with a command line argument. Here is the step-by-step process of my code:
1. The boss enqueues the number of requests to the worker threads that were just initialized by a command line argument. The work queue will implement `steque.[ch]`.
2. The worker threads continue to run.
3. The boss will terminate the worker threads once it is confirmed that all requests have been completed.
4. The program exits.

To coordinate the activities above, at least one condition variable `pthread_cond_t` will be used. Furthermore, at lease one mutex `pthread_mutex_t` will be employed as well.