# I/O

## **1. I/O Models Overview**
I/O operations are categorized into combinations of synchronous/asynchronous and blocking/non-blocking:

| **Model**                  | **Description**                                                                                                                                 | **Examples**                                         |
|----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------|
| **Synchronous Blocking**   | Thread blocks while waiting for I/O readiness and execution.                                                                                   | Traditional file I/O in many programming languages. |
| **Synchronous Non-blocking** | Thread does not block while waiting for readiness but performs the I/O operations itself.                                                      | `select`, `poll`, or `epoll` readiness notification |
| **Asynchronous Blocking**  | Not common. Combines asynchronous readiness with blocking I/O calls.                                                                           | Rarely implemented.                                 |
| **Asynchronous Non-blocking** | Operations are queued, and the thread is notified upon I/O completion.                                                                        | `io_uring`, Python’s `asyncio`.                  |

---

## **2. Event Loop**
An **event loop** is a core component of asynchronous frameworks. It:

- Monitors multiple I/O sources.
- Dispatches events to handlers when they’re ready.
- Typically built around OS-level APIs like `select`, `epoll`, or `io_uring`.

### **How Event Loops Work in Unix**
1. A demultiplexer (e.g., `select` or `epoll`) monitors I/O sources.
2. The loop waits for events and dispatches them to callbacks or handlers.
3. Handlers execute and optionally re-register for future events.

### **Python’s asyncio**
- Combines an event loop with coroutines for asynchronous programming.
- Implements the **Asynchronous Non-blocking I/O model**.
- Often integrates with `epoll` or equivalent APIs for readiness notifications.

---

## **3. Reactor Pattern**
The **Reactor Pattern** is a design pattern for event-driven I/O:

- **Core Components**:
  1. **Event Demultiplexer**: Monitors resources for readiness (e.g., `epoll`).
  2. **Dispatcher**: Maps events to corresponding handlers.
  3. **Handlers**: Execute application-specific logic.

- **I/O Model**: Asynchronous Non-blocking.
- Examples: Java NIO’s `Selector`, libraries like Netty.

---

## **4. Akka’s Actor Model vs. Reactor Pattern**
| **Aspect**            | **Reactor Pattern**                              | **Actor Model (Akka)**                    |
|-----------------------|-------------------------------------------------|-------------------------------------------|
| **Core Concept**      | Event-driven I/O handling via an event loop.     | Concurrency through message-passing actors. |
| **Concurrency Model** | Single-threaded event loop.                     | Message-passing between independent actors. |
| **Execution**         | One thread manages many connections.            | Actors scheduled with thread pools.        |

Akka’s Actor Model does **not** directly follow the Reactor Pattern. It emphasizes concurrency over I/O handling.

---

## **5. NIO and AIO in Java**
| **Feature**             | **NIO (Non-blocking I/O)**               | **AIO (Asynchronous I/O)**                |
|--------------------------|------------------------------------------|-------------------------------------------|
| **Pattern Used**         | Reactor                                 | Proactor                                  |
| **Thread Usage**         | Single-threaded with `Selector`.         | Uses completion handlers or `Future`.     |
| **Scalability**          | High but requires readiness polling.    | Higher due to true asynchronous handling. |

### **Key Differences**:
- **NIO**: Event-driven readiness notification (e.g., `Selector` API).
- **AIO**: Callback-based completion model with native OS support.

---

## **6. epoll vs. io_uring**
| **Feature**             | **epoll**                               | **io_uring**                              |
|--------------------------|------------------------------------------|-------------------------------------------|
| **I/O Model**           | Asynchronous Non-blocking(but I/O execution is sync)  | Fully Asynchronous                        |
| **Mechanism**           | Event notification (polling readiness). | Submission/completion queues.             |
| **Performance**         | High but syscall-heavy.                 | Higher with fewer syscalls.               |
| **Kernel Interaction**  | Requires explicit readiness polling.    | Kernel executes I/O and signals completion. |
| **Direct Storage Access** | No                                      | Yes                                       |
| **Pattern**             | Reactor                                 | Proactor                                  |

### **Use Cases**
- **epoll**: High-performance servers (e.g., web servers, databases).
- **io_uring**: High-throughput systems needing true async I/O (e.g., storage systems).

---

## **7. Reactor vs. Proactor**:
- Reactor (e.g., epoll, NIO): Monitors readiness and requires explicit I/O operations.  
  **The event loop signals readiness, and the thread performs the I/O synchronously.**
- Proactor (e.g., io_uring, AIO): Delegates both readiness and I/O execution to the kernel.  
  **The kernel or hardware directly performs the I/O operation and notifies the application only upon completion.**
  
Choose the appropriate model or library based on scalability, performance, and abstraction needs for your application.

---
## Q/A

1. What is I/O Readiness?
I/O readiness refers to a state where an I/O resource (e.g., a socket, file descriptor) is ready to perform a specific operation without blocking. For example:  
  A socket is read-ready if there is data available to be read.  
  A socket is write-ready if it can accept new data without blocking.  
  Operating systems provide mechanisms (like epoll or select) to notify applications when a resource becomes ready, avoiding unnecessary blocking or polling.

2. Clarifying "Thread does not block while waiting for readiness but performs the I/O operations itself"
In Synchronous Non-blocking I/O, the process works as follows:  
  The application uses non-blocking APIs to check I/O readiness (e.g., select, poll, or epoll).  
  If the resource is not ready, the call returns immediately without blocking the thread.  
  Once the resource is ready, the application performs the actual I/O operation (e.g., reading or writing).  
  The key idea is that the thread is not blocked while waiting for readiness but still actively executes the I/O operations (read/write) when readiness is signaled.