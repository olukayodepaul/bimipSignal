
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

// ---------------- Identity ----------------
message Identity {
    string eid = 1;                              // User’s global identifier (e.g., account ID or JID)
    optional string connection_resource_id = 2;  // Identifier for a specific device or session instance
    string node = 3;                             // System node handling this entity (cluster context)
}

// ---------------- Media ----------------
message Media {
    string type = 1;        // "image" | "video" | "audio" | "file"
    string url = 2;         // URL of the media
    string thumbnail = 3;   // Thumbnail URL (optional)
    int64 size = 4;         // Size in bytes
}

// ---------------- Signal ----------------
message Signal {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    int32 type = 4;       // 1=REQUEST, 2=RESPONSE, 3=ERROR
    int32 status = 5;     // 6=TYPING, 7=RECORDING, 8=FORWARDED, etc.
    int64 timestamp = 6;
    int64 monotonic_id = 7;
}

// ---------------- Payload ----------------
message Payload {
    map<string, string> data = 1;
    repeated Media media = 2;
}

// ---------------- Message ----------------
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

// ---------------- Awareness ----------------
message Awareness {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    int32 type = 4;
    int32 status = 5;
    int64 timestamp = 6;
}

// ... (other stanzas like Ack, ErrorMessage, TokenAuthority, etc.)
````

> 💡 **Note:**
> BIMIP stanzas are intentionally modular — you can extend the protocol to include custom message types such as **Offers**, **Candidates**, or **System Events** while maintaining binary compatibility.

---

## 🧱 Protocol Index

| **Message Type** | **Purpose**                                                                  |
| ---------------- | ---------------------------------------------------------------------------- |
| `Identity`       | Defines user and device identity within a cluster.                           |
| `Signal`         | Handles transient actions like typing, recording, or delivery notifications. |
| `Awareness`      | Communicates real-time presence and activity.                                |
| `Message`        | Represents chat or system notifications.                                     |
| `TokenAuthority` | Manages token refresh, revocation, or validation.                            |
| `Logout`         | Performs device or session logout.                                           |
| `ErrorMessage`   | Provides structured error responses.                                         |

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




