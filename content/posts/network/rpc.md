---
title: "Notes on RPC Framework Design and Implementation"
date: 2022-01-29T17:45:52+08:00
draft: false
categories: ["Network"]
description: "A translated technical note on Notes on RPC Framework Design and Implementation, preserving the examples and context of the original article."
---
# Notes on RPC Framework Design and Implementation

> Originally published in Chinese on 2022-01-29; this English edition preserves the original scope and technical context.

## Concept

RPC (Remote Procedure Call) is called Remote Procedure Call, which involves using a network to request services from a remote computer: it can be understood as placing part of a program on a remote computer to execute. Through network communication, a call request is sent to the remote computer, where the system resources of the remote computer are used to execute this part of the program, and finally returns the execution result to the remote computer.

Decompose the concept of Remote Procedure Call into "Remote Procedure" and "Procedure Call" to understand more intuitively:

Remote Procedure: A remote procedure is in contrast to a local procedure, which can also be referred to as a local function. A local procedure refers to the fact that the method that initiates the call and the method that is called are within the same address space or memory space. A remote procedure, on the other hand, involves placing part of the program logic from one process to another machine, which is commonly referred to as service decomposition, where each service is responsible for a single business, and each service has independent scalability and upgradeability, and is easy to maintain. The services provided on each machine are called remote procedures. This concept makes it easier to build distributed systems, laying the foundation for the service-oriented architectural style.

Process Invocation: This concept is very straightforward, encompassing method calls and function calls that we commonly see, and used for program control and data transmission. When "process invocation" encounters "remote process", it means that process invocation can span machines and networks for program control and data transmission.

## Selection

Choosing RPC Frameworks: Metrics and Considerations

Using RPC frameworks essentially boils down to three choices.

1. Self-designed RPC framework can start from deployment, aiming to create a suitable RPC framework that fits the business characteristics and scenarios. However, a self-designed framework requires substantial financial and human resources.

2. By modifying an open-source RPC framework, making it more suitable for the business scenario. Compared to the first approach, this method has lower human resource costs. However, this method requires keeping in sync with the open-source community for updates. If at some point the company's modified RPC framework stops synchronizing with the community, it may become incompatible with the latest community version at a certain point. This could eventually lead the company to revert to the first approach.
3. Use a fully open-source RPC framework and regularly sync with the community version. This choice minimizes the amount of human resources needed, and many issues can be resolved through community assistance. However, taking an open-source RPC framework directly and using it comes with many limitations. Each part of the framework is designed to elegantly solve business scenarios, rather than the reverse. Additionally, RPC frameworks have their own positioning and future plans, so most medium-sized and larger companies opt to make some modifications to the framework to better suit their specific business scenarios.



## Java I/O Model Enveloping

### NIO
In Java NIO, the core is the selector. Whenever one of the connection events, accept connection events, read events, and write events is ready, the corresponding event handler executes its logic. This event-driven model is called the Reactor model. The core idea of the Reactor model is to reduce thread waiting. When facing an IO operation that requires waiting, resources are released first, and then, upon completion of the IO operation, the processing continues through event-driven mechanisms. This way, resources are consumed more efficiently in the overall context.

Here are the five important roles of the Reactor pattern:

- Handle (descriptors on Linux): It is an abstraction of a resource at the operating system level, representing a resource provided by the operating system, such as socket descriptors in network programming or file descriptors. This resource is bound to an event, and can also represent individual events, such as connection events from a client, reception connection events from a server, and write data events.
- Synchronous Event Demultiplexer: handles registered events and dispatches them when they become ready. In essence, it is a system call that waits for events. The caller remains blocked until the demultiplexer has a ready event. Linux provides `select`, `poll`, and `epoll`; Java NIO exposes the corresponding abstraction as `Selector`, with `select` as the blocking method.
- Event Handler (Event Handler): It consists of multiple callback methods, which are the logical responses to a specific event. Event Handlers are usually abstract interfaces. For example, callback methods for when a Channel registers with a Selector, connection events, and write events are all Event Handlers. We can implement these callback methods to achieve specific feedback for a particular event. In Java NIO, there is no abstract provided for Event Handlers.
- Concrete Event Handler (Concrete Event Handler): It implements the Event Handler. It implements the specific business logic by providing various callback methods defined by the Event Handler. For example, to log a message when a connection event occurs, the logging logic can be implemented within the connection event's callback method.
- Initiation Dispatcher (Initial Dispatcher): Considered as a Reactor, it specifies the scheduling strategy for events and manages event handlers, providing methods for registration and deregistration. Event handlers must be registered on the Initial Dispatcher to become effective. It serves as the core of all event handlers. The Initial Dispatcher uses a Synchronous Event Demultiplexer to wait for events. Once an event occurs, the Initial Dispatcher first separates it, then locates the corresponding event handler, and finally invokes the related callback methods to process these events.



Three Models of the Reactor

Single Reactor Single Thread Model

Single Reactor Single-Thread Model designs with only one Reactor, where both I/O-related read/write operations and I/O-independent encoding/decoding or computation are handled in a single Handler thread.

#### Single Reactor Multi-thread Model

In the multi-threaded model of a single Reactor, the threads are used to execute logic unrelated to IO operations. IO reads/writes and Reactor processing are handled by a single thread.

Against the first model, delegating business logic to a thread pool can fully leverage the processing capability of multi-core CPUs.
Force, but Reactor handles all event listening and response with a single thread, which can lead to performance bottlenecks in high-concurrency scenarios. Thus, the master-slave Reactor multi-threaded model was introduced.

#### Master-Slave Reactor Multi-threaded Model

When the number of client connections is high and IO operations are frequent, a single Reactor can expose issues. Since a single Reactor can only handle IO operations synchronously, connection events are often less frequent than read/write events. When the Reactor is busy handling read/write events, new client connection events are blocked, which can lead to connection timeouts and other issues.

In the master-slave Reactor multi-threaded model, this problem is addressed. The thread responsible for handling connection events is isolated from the thread handling read/write events, avoiding the issue of blocking new client connections due to frequent read/write events. In the master-slave Reactor multi-threaded model, there are multiple Reactors, with the Main Reactor typically being a single instance responsible for listening and handling connection requests. The Sub Reactors, managed by a thread pool, handle read/write events and other tasks.

### AIO (Asynchronous I/O)

AIO stands for Asynchronous I/O, introduced in JDK 1.7 as an enhanced Java I/O class library. It provides an asynchronous and non-blocking IO operation model. Asynchronous IO is achieved through event and callback mechanisms. After an application initiates a request, the thread executing the request does not block; instead, the operating system notifies the corresponding thread when the background processing is complete. AIO has two usage styles:

- Future Style: Java uses the Future class for the Future style. After submitting a task to a thread pool, the thread executing the task does not block. Instead, it returns a Future object. The task's return value can be obtained from the Future object once the task is completed. However, the result can only be retrieved by calling the `get()` method method, which can lead to blocking if the result is not available. This approach can be less efficient than synchronous calls.
- Callback Style: Java provides the `CompletionHandler` interface for callbacks. When calling methods `read`, `write`, etc., a `CompletionHandler` implementation can be passed as the event completion callback. This requires the user to implement the business logic for the callback.

plaintext

## Protocol

### Custom Protocol

#### Advantages

When designing a system, the selection or design of protocols is crucial. Compared to HTTP, a widely-used standard protocol, the advantage of a custom protocol lies in:

1. **Good scalability**. The system needs to evolve and iterate. If future evolution plans involve changes at the protocol level, a less scalable protocol will hinder system evolution. Custom protocols can be designed for scalability, meeting the need to expand according to business requirements and development. Standard protocol HTTP was designed for generality and is difficult to extend.
2. **Higher security**. The data format of custom protocols is transparent and open, thereby enhancing communication security and encrypting data to improve the security of transmitted data.
3. **Higher Transmission Efficiency**. The format of the protocol itself affects the size and speed of data packets carried by a single request. A custom protocol can be designed to be efficient and suitable for the system at hand. However, the scalability and efficiency of a custom protocol sometimes conflict, requiring careful consideration based on the actual scenario. In general, a custom protocol offers greater flexibility but is more complex to design and implement. If the system scenario is simple, a custom protocol can actually increase system design complexity and extend the development cycle. The benefits of a custom protocol may not be significant.

#### Steps

1. **Clearly define the two parts that need to be designed in the communication protocol**: header and body. The header can be seen as the part carrying special protocol fields, while the body is the actual data entity to be transmitted.
2. **Necessary fields for the header design**. When designing the necessary fields for the header, the following necessary fields (field names are not important and can be changed arbitrarily) need to be considered:
- `version`: Here, the version number refers to the protocol version number. Including this field in the data packet allows the server to know which version of the protocol the client is using upon parsing the data packet. This enables the server to implement compatibility measures.
This mechanism is extremely flexible and perfectly addresses the compatibility issues between different version protocols. For example, HTTP, in addition to the standard protocol, has evolved into other protocols based on HTTP, such as the WebSocket protocol. <br>
3. The third step is to determine the encoding method for the protocol header. One method is binary encoding, and the other is text encoding. <br>
4. The fourth step is to determine the arrangement order of each data bit and each field.
