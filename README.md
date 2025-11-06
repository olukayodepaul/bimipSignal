
# 🛰️ BIMIP — Binary Interface for Messaging and Internet Protocol

**BIMIP** is a scalable real-time communication protocol built for developers who need **reliable**, **low-latency** connectivity across devices, users, and distributed systems.

It is designed for **massive-scale chat**, **location streaming**, **push notifications**, **chat message awareness**, **media streaming**, and **real-time collaboration** — all powered by a **binary, event-driven architecture** that maximizes performance and efficiency.

Built on the **BEAM (Erlang/Elixir)** runtime, BIMIP provides **fault tolerance**, **massive concurrency**, and **distributed scalability** out of the box — making it ideal for **mission-critical, always-connected systems**.

At its core, **BIMIP** is composed of two primary services:

* **BimipSignal** — Handles `BimipClient` connections, signaling, and message serialization/deserialization.
* **BimipServer** — Serves as the **authoritative control plane** for identity, token management, contact coordination, and inter-device routing.

Together, they form a **robust, distributed communication layer** that ensures:

* ⚡ **High throughput** for millions of concurrent sessions
* 🚀 **Low-latency routing** across nodes and regions
* 🧭 **Persistent awareness** of user and device presence
* 🔄 **Seamless synchronization** between multiple connected devices

---

## 🧩 Core Components

| **Component**                     | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **BimipServer**                   | Core router responsible for managing sessions, message routing, and device awareness events across clusters.                                                                                                                                                                                                                                                                                                                                        |
| **BimipSignal**                   | Handles client-side signaling, message serialization, and event propagation for connected `BimipClient` instances.                                                                                                                                                                                                                                                                                                                                  |
| **Epokhai Session Manager (ESM)** | Coordinates and manages device communication across distributed nodes, maintaining per-device awareness and state.                                                                                                                                                                                                                                                                                                                                  |
| **Orchestrator**                  | Supervises and coordinates node clusters for scalability, recovery, and load balancing.                                                                                                                                                                                                                                                                                                                                                             |
| **Registry**                      | Tracks connected **EIDs** and **device IDs** across clusters using the **BIMIP Internal Presence Protocol (BIPP)** for real-time visibility.                                                                                                                                                                                                                                                                                                        |
| **Storage / BimipQueue**          | High-performance file-backed log system for persisting messages, acknowledgments, and device states.<br><br>Implements **sparse indexing**, **write-ahead logging**, and **segmented file design**, supporting ultra-fast append operations, offline persistence, and efficient cleanup of expired segments.<br><br>This guarantees **millions of messages** can be stored and processed efficiently with **minimal disk I/O** and **low latency**. |

---

## ✨ Features

* **Binary Protocol** — Compact, efficient payloads optimized for low-latency communication
* **Device Awareness** — Real-time, multi-device presence tracking
* **Scalable Routing** — High-performance routing using elixir OTP
* **Cluster-Ready Architecture** — Supports node interconnection via RPC or gRPC for distributed scalability
---

## 📜 Protocol Specification

```proto
syntax = "proto3";

package bimip;

/*
  ======================================================
  Bimip — Binary Interface for Messaging and Internet Protocol
  ======================================================

  Features:
  * Binary protocol for compact payloads
  * Device awareness (multi-device presence tracking)
  * Scalable routing using elixir OTP
  * Cluster-ready (node interconnection via RPC)

  ------------------------------------------------------
  Message Stanzas
  ------------------------------------------------------
  | Message Type     | Purpose                                                                 |
  | ---------------- | ----------------------------------------------------------------------- |
  | Identity         | Defines user and device identity within a cluster.                      |
  | Signal           | Handles transient actions (typing, recording, delivery notifications).   |
  | Awareness        | Communicates real-time presence and activity.                            |
  | Message          | Represents chat or system notifications.                                 |
  | TokenAuthority   | Manages token refresh, revocation, or validation.                        |
  | Logout           | Performs device or session logout.                                       |
  | ErrorMessage     | Provides structured error responses.                                     |
*/

// ---------------- Identity ----------------
message Identity {
    string eid = 1;                              // User’s global identifier (e.g., account ID or JID)
    optional string connection_resource_id = 2;  // Identifier for a specific device/session
    string node = 3;                             // Node handling this entity (cluster context)
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
// Represents a chat or notification message between users.
// Message-level ACKs are tracked via the status field.
// Awareness (TYPING, RECORDING) is per-user and handled separately.
// This is sent once.
message Message {
  string id = 1;              // client/queue-generated message ID
  string signal_offset = 2;   // server-assigned global ID (monotonic or unique)
  string user_offset = 3;     // server-assigned global ID (monotonic or unique)
  Identity from = 4;          // sender (JID or user ID)
  Identity to = 5;            // recipient (JID or user ID)
  int32 type = 6;             // 1 = Chat | 2 = PushNotification
  int64 timestamp = 7;        // epoch milliseconds
  bytes payload = 8;          // pass JSON { }
  string encryption_type = 9; // "none", "AES256", "E2E", etc.
  string encrypted = 10;       // base64 encrypted content (if any)
  string signature = 11;      // base64 signature for integrity
  int32 status = 12;          // 1=SENT, 2=RECORDING, 3=DELIVERED, 4=READ, 5=FORWARDED, 6=PLAYED/VIEWED,
                              // 7=TYPING, 8=RECORDING, 9=PAUSED, 10=CANCELLED, 11=RESUME,
                              // 12=CALLING, 13=DECLINE
  int32 signal_type = 13;     // 1=ONE-WAY, 2=TWO-WAY
}

// ---------------- Signal ----------------
// Used by both client and server to reconcile message status.
// Supports dual reference (id + signal_id) for precise synchronization.
message Signal {
  string id = 1;            // client/queue message ID
  string signal_offset = 2; // server-assigned global ID (monotonic or unique)
  string user_offset = 3;   // server-assigned global ID (monotonic or unique)
  int32 status = 4;         // 1=SENT, 2=RECORDING, 3=DELIVERED, 4=READ, 5=FORWARDED, 6=PLAYED/VIEWED, 7=TYPING, 8=RECORDING, 9=PAUSED, 10=CANCELLED, 11=RESUME
  int64 timestamp = 5;      // epoch ms
  Identity from = 6;        // who sent the ACK
  Identity to = 7;          // who receives the ACK
  int32 type = 8;           // 1=REQUEST, 2=RESPONSE, 3=ERROR
  int32 signal_type = 9;    // 1=ONE-WAY, 2=TWO-WAY
  optional string error = 10;// optional error message if type=3
}


// ---------------- PushNotification ----------------
message PushNotification {
    string id = 1;
    Identity from = 2;
    Identity to = 3;
    string type = 4;
    int64 timestamp = 5;
     bytes payload = 8;                // pass json { }
    Ack ack = 7;
    Metadata metadata = 8;
}

// ---------------- ErrorMessage ----------------
message ErrorMessage {
    int32 code = 1;
    int32 error_origin = 2;
    string details = 3;
    int64 timestamp = 4;
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
    int32 task = 4;
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
    int32 type = 2;       // 1=DEVICE, 2=SESSION, 3=ACCOUNT
    int32 status = 3;     // 0=INITIATED, 1=SUCCESS, 2=FAILED
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
```

---

## 🧱 Protocol Index

| **Message Type**       | **Purpose**                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| `Identity`              | Defines user and device identity within a cluster, including node and session context.               |
| `Media`                 | Describes attached media (image, video, audio, or file) with optional thumbnail and metadata.        |
| `Signal`                | Handles transient actions such as typing, recording, forwarding, delivered, or read notifications.   |
| `Payload`               | Encapsulates message content and metadata, including optional media attachments.                     |
| `Metadata`              | Provides encryption and signature for message integrity and confidentiality.                         |
| `Ack`                   | Tracks acknowledgment states (SENT, DELIVERED, READ, FORWARDED, PLAYED).                             |
| `Awareness`             | Communicates real-time presence, device activity, and location sharing.                              |
| `Message`               | Represents chat messages or system notifications with acknowledgment tracking.                       |
| `PushNotification`      | Delivers push messages to devices, supporting offline and background notification delivery.           |
| `ErrorMessage`          | Defines structured error responses with standardized codes and details.                              |
| `PingPong`              | Maintains heartbeat (PING ↔ PONG) between clients and servers.                                       |
| `Contact`               | Manages relationships and subscription actions (add, remove, accept, deny, etc.).                    |
| `AwarenessVisibility`   | Toggles user visibility state (ENABLED, DISABLED, or ERROR).                                         |
| `TokenAuthority`        | Manages token operations (REQUEST, REFRESH, VALIDATE, or REVOKE).                                    |
| `LocationStream`        | Streams real-time geolocation updates including latitude, longitude, and optional altitude data.      |
| `Logout`                | Performs device or session logout with status reporting.                                             |
| `Body`                  | Container for batching multiple stanzas (supports Awareness lists).                                  |
| `MessageScheme`         | Unified wrapper for all stanza types ensuring one payload per transport message.                     |

---

## 🧠 Ideal Use Cases

* Real-time chat or collaboration apps
* Video/audio call signaling
* IoT device coordination
* Distributed event streaming
* Multi-device synchronization

---


## 📄 License

Released under the [MIT License](./LICENSE)

---

## 👨‍💻 Author

**Bimip Foundation**
📧 [paul.olukayode.pro@gmail.com](mailto:paul.olukayode.pro@gmail.com)
🌐 [GitHub](https://github.com/olukayodepaul) 
• [LinkedIn](https://linkedin.com/in/paulaigbokhaiolukayode)

