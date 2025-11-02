
# 🛰️ BIMIP — Binary Interface for Messaging and Internet Protocol

**BIMIP** is a scalable real-time communication protocol built for developers who need reliable, low-latency connectivity across devices, users, and distributed systems.

It is designed for **massive-scale chat**, **location streaming**, **push notifications**, **chat message awareness**, **media streaming**, and **real-time collaboration** — all powered by a **binary, event-driven architecture** that maximizes performance and efficiency.

Built on the **BEAM (Erlang/Elixir)** runtime, BIMIP delivers **fault tolerance**, **massive concurrency**, and **distributed scalability** out of the box — making it ideal for mission-critical, always-connected applications.

At its core, **BIMIP** is composed of two main services:

- **BimipSignal** — Handles **BimipClient** connections, signaling, and message serialization/deserialization.  
- **BimipServer** — Serves as the **authoritative control plane**, where contact lists are managed (add, delete, block), tokens are signed, and inter-device coordination is enforced.

Together, they form a **robust communication layer** that ensures:

- ⚡ **High throughput** for millions of concurrent sessions  
- 🚀 **Low-latency routing** across nodes and regions  
- 🧭 **Persistent awareness** of user and device presence  
- 🔄 **Seamless synchronization** between multiple connected devices  

---

## 🚀 Vision

> “To create a universal, binary-based communication interface that connects millions of devices efficiently and securely.”

**BIMIP (Binary Interface for Messaging and Internet Protocol)** provides an open framework for building **high-performance message brokers** and **device-aware systems** that can operate globally in a distributed environment.

---

## 🧩 Core Components

| **Component**                     | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **BimipServer**                   | Core router managing sessions, message routing, and awareness events.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **BimipSignal**                   | Handles **BimipClient** connections, signaling, and message serialization/deserialization.                                                                                                                                                                                                                                                                                                                                                                                             |
| **Epokhai Session Manager (ESM)** | Manages and coordinates device communication across nodes.                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Orchestrator**                  | Supervises and coordinates distributed node clusters for scalability and fault tolerance.                                                                                                                                                                                                                                                                                                                                                                                              |
| **Registry**                      | Tracks connected **EIDs** and **device IDs** across clusters using the **BIMIP Internal Presence Protocol (BIPP)**.                                                                                                                                                                                                                                                                                                                                                                    |
| **Storage**                       | The **BimipQueue** is a high-performance, file-backed log system responsible for persisting messages, acknowledgments, and device states.<br><br>Built on advanced principles of **sparse indexing**, **write-ahead logging**, and **segmented file design**, it supports fast append operations, offline data persistence, and efficient purging of old segments.<br><br>This design ensures scalability for millions of messages while maintaining low latency and minimal disk I/O. |

---

## ✨ Features

- **Binary Protocol** — Compact, efficient payloads optimized for low-latency communication  
- **Device Awareness** — Real-time multi-device presence tracking across sessions  
- **Scalable Routing** — High-performance message routing using ETS and GenServers  
- **Cluster-Ready Architecture** — Supports node interconnection via RPC or gRPC for distributed scalability  
- **Extensible Message Schema** — Flexible structure for offers, ICE candidates, and custom event types  

---

## 🧩 Protocol Definition

BIMIP uses **Protocol Buffers (proto3)** as its core serialization format.  
The `.proto` specification defines all communication stanzas — from messages and awareness signals to authentication and session control.

Below is the current BIMIP protocol schema:

```proto
syntax = "proto3";

package bimip;
````
````proto
message Identity {
    string eid = 1;    
    optional string connection_resource_id = 2; 
    string node = 3;
}
````

````proto
message Media {
    string type = 1;  
    string url = 2; 
    string thumbnail = 3;
    int64 size = 4;  
}
````
````proto
message Signal {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    int32 type = 4;
    int32 status = 5;
    int64 timestamp = 6;
    int64 monotonic_id = 7;
}
````
````proto
message Payload {
    map<string, string> data = 1;
    repeated Media media = 2;
}
````
````proto

message Message {
    string id = 1;
    string from = 2;
    string to = 3;
    string type = 4;
    int64 timestamp = 5;
    Payload payload = 6;
    Ack ack = 7;
    Metadata metadata = 8;
}
````

````proto
message Awareness {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    int32 type = 4;
    int32 status = 5;
    int64 timestamp = 6;
}
````

---

## 🧱 Protocol Index

| **Message Type**          | **Purpose**                                                                                                     |
|----------------------------|-----------------------------------------------------------------------------------------------------------------|
| `Identity`                | Defines user and device identity within a cluster, including node association and session context.              |
| `Media`                   | Describes attached media (image, video, audio, or file) with optional thumbnail and metadata.                   |
| `Signal`                  | Handles transient events like typing, recording, forwarded, delivered, read, and resume notifications.          |
| `Payload`                 | Encapsulates message data and key-value attributes, including optional media attachments.                       |
| `Metadata`                | Provides encryption and signature fields for message integrity and confidentiality.                             |
| `Ack`                     | Tracks acknowledgment states (SENT, DELIVERED, READ, FORWARDED, PLAYED) for reliable delivery.                  |
| `Awareness`               | Communicates real-time presence, device activity, and location sharing with TTL-based expiration.               |
| `Message`                 | Represents chat messages or notifications with payload, metadata, and acknowledgment tracking.                  |
| `ErrorMessage`            | Defines structured error responses with standardized codes and contextual details.                              |
| `PingPong`                | Maintains connection heartbeat between client and server (PING ↔ PONG).                                         |
| `Contact`                 | Manages contact relationships and subscription actions (add, remove, accept, deny, etc.).                       |
| `AwarenessVisibility`     | Toggles and synchronizes user awareness visibility state (ENABLED, DISABLED, or ERROR).                         |
| `TokenAuthority`          | Manages token operations such as REQUEST, RESULT, REFRESH, or REVOKE between nodes or clients.                  |
| `Logout`                  | Performs device or session logout with status reporting (SUCCESS, FAIL, PENDING).                              |
| `Body`                    | Container for batching multiple stanzas (currently supports Awareness lists).                                   |
| `MessageScheme`           | Unified wrapper for all stanza types, ensuring only one payload type per transport message.                     |


---

## 🧠 Ideal Use Cases

* Real-time messaging apps
* Video/audio call signaling
* IoT device coordination
* Distributed event streaming
* Multi-device synchronization

---

## 🧭 Documentation

All details about architecture, protocol design, and setup are in the [`docs/`](./docs) directory:

* [Overview](./docs/1-overview.md)
* [Architecture](./docs/2-architecture.md)
* [Protocol Specification](./docs/3-protocol-spec.md)
* [Message Flow](./docs/4-message-flow.md)
* [Setup Guide](./docs/5-setup.md)
* [Use Cases](./docs/6-use-cases.md)
* [Glossary](./docs/glossary.md)

---

## 📄 License

Released under the [MIT License](./LICENSE)

---

## 👨‍💻 Author

**Paul Aigokhai Olukayode**
Full Stack Software Engineer • Architect of BIMIP
📧 [paul.olukayode.pro@gmail.com](mailto:paul.olukayode.pro@gmail.com)
🌐 [GitHub](https://github.com/olukayodepaul) | [LinkedIn](https://linkedin.com/in/paulaigbokhaiolukayode)




