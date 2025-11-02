
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

syntax = "proto3";

package bimip;

  ======================================================
  Bimip — Binary Interface for Messaging and Internet Protocol
  ======================================================

  Features:
  * **Binary protocol** for compact payloads
  * **Device awareness** (multi-device presence tracking)
  * **Scalable routing** using ETS + GenServers
  * **Cluster-ready** (node interconnection via RPC or gRPC)
  * **Extensible message schema** for offers, candidates, and custom events

  ------------------------------------------------------
  Message Stanzas
  ------------------------------------------------------
  | **Message Type** | **Purpose**                                                                  |
  | ---------------- | ---------------------------------------------------------------------------- |
  | `Identity`       | Defines user and device identity within a cluster.                           |
  | `Signal`         | Handles transient actions like typing, recording, or delivery notifications. |
  | `Awareness`      | Communicates real-time presence and activity.                                |
  | `Message`        | Represents chat or system notifications.                                     |
  | `TokenAuthority` | Manages token refresh, revocation, or validation.                            |
  | `Logout`         | Performs device or session logout.                                           |
  | `ErrorMessage`   | Provides structured error responses.                                         |
*/

// ---------------- Identity ----------------
message Identity {
    string eid = 1;                              // User’s global identifier (e.g., account ID or JID)
    optional string connection_resource_id = 2;  // Identifier for a specific device/session
    string node = 3;                             // Node handling this entity (cluster context)
}

// ---------------- Media ----------------
message Media {
    string type = 1;        // "image" | "video" | "audio" | "file"
    string url = 2;         // Media URL
    string thumbnail = 3;   // Optional thumbnail
    int64 size = 4;         // Size in bytes
}

// ---------------- Signal ----------------
message Signal {
    string id = 1;                      // Unique signal ID (UUID or correlation reference)
    Identity from = 2;                  // Source entity
    Identity to = 3;                    // Target entity
    int32 type = 4;                     // 1=REQUEST, 2=RESPONSE, 3=ERROR
    int32 status = 5;                   // 1=TYPING, 2=RECORDING, 3=FORWARDED, 4=DELIVERED, 5=READ, 6=RESUME
    int64 timestamp = 6;                // Epoch milliseconds
    int64 monotonic_id = 7;             // Derived sequence number from message queue
}

// ---------------- Payload ----------------
message Payload {
    map<string, string> data = 1;   // Dynamic key-value data
    repeated Media media = 2;       // Optional media attachments
}

// ---------------- Metadata ----------------
message Metadata {
    string encrypted = 1;    // Base64 encrypted content
    string signature = 2;    // Base64 signature for integrity
}

// ---------------- Ack ----------------
message Ack {
    repeated int32 status = 1; // 9=DELIVERED, 10=READ, 11=FORWARDED, 12=SENT, 13=PLAYED/VIEWED
}

// ---------------- Awareness ----------------
message Awareness {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    int32 type = 4;                 
    int32 status = 5;               
    int32 location_sharing = 6;     
    double latitude = 7;
    double longitude = 8;
    int32 ttl = 9;
    string details = 10;
    int64 timestamp = 11;
    int32 visibility = 12;
}

// ---------------- Message ----------------
message Message {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    string type = 4;
    int64 timestamp = 5;
    Payload payload = 6;
    Ack ack = 7;
    Metadata metadata = 8;
}

// ---------------- PushNotification ----------------
message PushNotification {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    string type = 4;
    int64 timestamp = 5;
    Payload payload = 6;
    Ack ack = 7;
    Metadata metadata = 8;
}

// ---------------- ErrorMessage ----------------
message ErrorMessage {
    int32 code = 1;           // e.g., 400=Bad Request, 401=Unauthorized
    int32 error_origin = 2;   // e.g., 1=CLIENT, 2=SERVER, 3=NETWORK
    string details = 3;       // Human-readable description
    int64 timestamp = 4;      // Error occurrence timestamp
}

// ---------------- PingPong ----------------
message PingPong {
    string id = 1;
    Identity from = 2;
    int32 type = 3;           // 1=PING, 2=PONG
    int64 timestamp = 4;
    string details = 5;
}

// ---------------- Contact ----------------
message Contact {
    Identity from = 1;
    Identity to = 2;
    string tracking_id = 3;
    int32 relationship = 4;
    int32 action = 5;
    int64 timestamp = 6;
    string details = 7;
}

// ---------------- AwarenessVisibility ----------------
message AwarenessVisibility {
    string id = 1;
    Identity from = 2;
    int32 type = 3;
    int64 timestamp = 4;
    string details = 5;
}

// ---------------- TokenAuthority ----------------
message TokenAuthority {
    Identity to = 1;
    string token = 2;
    int32 type = 3;        // 1=REFRESH, 2=VALIDATE, 3=REVOKE
    int32 task = 4;        // Specific action to perform
    int64 timestamp = 5;
    string details = 6;
}

// ---------------- LocationStream ----------------
message LocationStream {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    double latitude = 4;
    double longitude = 5;
    optional double altitude = 6;
    int64 timestamp = 7;
}

// ---------------- Logout ----------------
message Logout {
    Identity to = 1;
    int32 type = 2;         // 1=DEVICE, 2=SESSION, 3=ACCOUNT
    int32 status = 3;       // 0=INITIATED, 1=SUCCESS, 2=FAILED
    int64 timestamp = 4;
    string details = 5;
}

// ---------------- Body ----------------
message Body {
    int64 route = 1;
    repeated Awareness awareness_list = 2;
    int64 timestamp = 3;
}

// ---------------- MessageScheme ----------------
message MessageScheme {
    int64 route = 1;

    oneof payload {
        Awareness awareness = 2;
        PingPong ping_pong = 3;
        AwarenessVisibility awareness_visibility = 4;
        TokenAuthority token_authority = 5;
        Message chat_message = 6;
        PushNotification push_notification = 7;
        LocationStream location_stream = 8;
        Body body = 9;
        ErrorMessage error = 10;
        Logout logout = 11;
        Signal signal = 12;
    }
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




