---
description: A primer on how P2P messaging works in the Blockchain Development Kit (BDK).
---

# P2P Overview

This subchapter provides a comprehensive overview of the P2P classes and their organization within the BDK. It further elaborates on the life-cycle of a P2P connection and the dynamic flow of data between nodes. All classes described here are inside the `src/net/p2p` folder. Check their header files for full details on how each of them work.

## NodeConns

The `NodeConns` class (`nodeconns.h`) takes the role of maintaining and periodically updating a list of all peer nodes in the network that are connected to the local node, as well as their respective info. It is through this list that the node keeps itself up-to-date with the most recent node info possible, while also dropping stale connections/timed out peers every now and then to avoid potential network sync problems.

## Session

The `Session` class (`session.h`) encapsulates a TCP connection with a remote node, responsible for managing handshakes, sending and receiving messages by reading and writing socket data. It servees for both client and server connections, having queues for both inbound and outbound messages, allowing any thread that is responsible for sending a message to carry on with its task without having to wait for the message transmission to complete.

Sessions are designed to be used as `shared_ptr`s with `shared_from_this` to manage their lifecycles, as they primarily exist within the handlers of Boost's `io_context`. Once a session has successfully established a connection with a remote node, it is added to a list of sessions controlled by a manager. It is critical to properly manage those shared pointers, as the destructor of the `Session` class is intrinsically linked to the destructor of the socket, which, in turn, calls the `io_context`. If the pointer persists but its referenced `io_context` no longer exists, this will lead to a program crash.

For instance, functions called by the thread pool to parse asynchronous operations accept a `weak_ptr` as a parameter, which is then converted into a `shared_ptr`. This design choice ensures the `Session` object is still viable. In addition, the manager session mutex is locked to prevent the `shared_ptr` from unexpectedly becoming a `unique_ptr`, which would cause a segmentation fault while attempting to delete a session in the absence of its corresponding `io_context`.

## Life cycle of a session

Upon instantiation, a session's life-cycle is composed of three different routines: *handshake*, *read* and *write*.

### Handshake routine

The first routine is giving a handshake to a remote endpoint, effectively making a connection so data can be shared between both ends. It happens like this:

* The `run()` method is called first, and uses `boost::asio::dispatch()` to dispatch a call of `write_handshake()`
* `write_handshake()` uses `boost::asio::async_write()` to send a handshake to the remote endpoint, and sets `read_handshake()` as a callback for when the handshake is properly answered
* `read_handshake()` initiates an asynchronous read operation to receive the remote enpoint's handshake, and sets `finish_handshake()` as a callback for when the operation is complete
* `finish_handshake()` validates the recived handshake through the P2P manager and calls `do_read_header()` to start the next routine

### Read routine

The second routine is reading inbound messages coming from the remote endpoint. This routine is executed cyclically through the entire duration of the connection, and goes like this:

* `do_read_header()` reads an incoming message's header (8 bytes that contain the message's full size), and calls `on_read_header()` when done
* `on_read_header()` is a callback for when the header read process is complete, calling `do_message()` next
* `do_read_message()` reads the full message, using the size in bytes passed to it by `on_header()`, and calls `on_read_message()` when done
* `on_read_message()` is a callback for when the message read process is complete, parsing the message passed by `do_read_message()`, then calling `do_reaD_header()` once again and starting the next round of data exchange

### Write routine

The third routine is writing outbound messages going to the remote endpoint, like this:

* An external thread calls `write()` with a shared pointer to a message. Within this function, the queue mutex is locked, and a check is made to see if the outbound message is null
  * If it is null, the outbound message is assigned to the message passed as an argument, and `boost::asio::post()` is called with `do_message_header()` as its handler, with the appropriate write strand
  * If it is _not_ null (which indicates a write process is already going on), the message is added to the queue, and the write strand will handle it later
* The write strand then begins the following chain of actions:
  * `do_write_header()` writes the message's header (8 bytes that contain the message's full size), and calls `on_write_header()` when done
  * `on_write_header()` is a callback for when the header write process is done, calling `do_write_message()` next
  * `do_write_message()` writes the full message, and calls `on_write_message()` when done
  * `on_write_message()` is a callback for when the message write process is done, locking the queue mutex, grabbing the next message from the queue, and calling `do_write_header()` again - this loop goes on until the queue is empty, where the outbound message pointer is set to null and any further callbacks from the write strand come to a halt

## ManagerBase and derivatives

The P2P manager acts as the backbone of the P2P network. It bears the responsibility of managing `Session`s, overseeing their operations and indirectly managing the `io_context`, and serves as a base for the `ManagerNormal` (for Normal nodes) and `ManagerDiscovery` (for Discovery nodes) specialized classes.

Once a `Session` has successfully completed a handshake, it is registered within the manager, which then oversees the `Session`'s lifecycle. The manager's responsibilities include maintaining a registry of active `Session`s, handling incoming and outgoing requests and responses, and maintaining the communication between them.

Given its extensive duties, it's imperative that the functions within the manager remain as "lightweight" as possible within their respective mutexes. This is to ensure the manager is not locked for extended periods, as it is concurrently accessed by multiple threads to (de)register `Session`s, parse incoming messages, and/or request information from other nodes. If the manager stays locked for a long time, the node risks being blocked altogether, with potential repercussions extending to the entire network.

It's also important to be aware of the lifespan of the `io_context` and the objects that use it, before adding or modifying any code that uses the manager. `Session`s do not manage their `io_context`, which means that if you retrieve a `shared_ptr` to a specific `Session`, perform operations on it, and that pointer is deleted elsewhere, you could end up triggering an exception if using the now-dangling pointer because the `io_context` it refers to has already been destroyed.

## Message types

Every incoming message is promptly parsed by the manager via the `handleMessage()` function. These messages can fall into one of the following categories, with each one of them being treated distinctly:

* `Request` - a query for specific data from another node - e.g. a list of blocks, a list of transactions, info about the node itself, etc.
* `Answer` - an answer to a given request
* `Broadcast` - a dissemination of specific data to the network *with* possible re-broadcasting ("flooding"), such as a new block or transaction
* `Notification` - a dissemination of specific data to the network *without* re-broadcasting, such as node info

Both `Request` and `Answer` messages work together in a bidirectional flow that goes like this:

* The sender node initiates a `Request` by generating a random 8-byte ID, registering it internally and sending it alongside the message
* The receiver node receives the `Request`, its manager processes it by invoking the corresponding function to address it, formulates an `Answer` with the requested data, assigns it the same ID and sends it back to the sender node
* The sender node receives the `Answer` and checks if the received ID is the same one that was registered earlier. If it is, the manager fulfills the associated `Request` future with the received `Answer` and deregisters the ID. If the ID is _not_ registered, the `Answer` is discarded altogether

Both `Notification` and `Broadcast` messages, however, work with a simpler unidirectional flow, as the receiver node doesn't have to answer back to the sender. Instead, it verifies the received data and adds it to its own blockchain. The difference between both types is that `Notification` messages are never re-broadcast, while `Broadcast` messages may or may not be re-broadcast to other nodes, depending on whether said nodes had already received or not said broadcast in the past.

Due to this specific condition, `Broadcast` messages are specifically handled by the `Broadcaster` class (`Broadcaster.h`), while the other types are handled by `ManagerBase` and its derivative classes.

## Asynchronous Message Parsing

The first thread under our control that accesses the message is the `io_context` executing that particular `Session`. To optimize the performance of `io_context` and avoid any bottleneck, we offload the task of message parsing to a separate thread pool. This pool handles both parsing of the message and writing back to the `Session`, which involves adding tasks to its write strand or queue.

One performance-enhancing strategy we employ is the use of `shared_ptr` for handling each message. This prevents unnecessary copying and provides significant benefits when dealing with broadcasts. In such cases, a single message can be utilized by multiple writing sessions, thereby offering a performance boost to the network.

Our current design uses `shared_ptr` for all messages, however, we plan to transition to a system where `unique_ptr` is used for inbound messages and `shared_ptr` for outbound messages. The rationale behind this is to better handle memory ownership. The use of `unique_ptr` provides clearer ownership semantics and improved performance. Since `unique_ptr` cannot be moved into a `std::function` (which would be the task posted to the thread pool), we are simply using `shared_ptr` for now.

`handleAnswer()` always fills a `std::promise` with the answer. A thread that called a request within the manager towards another node will wait for a few seconds or until the answer is received.
