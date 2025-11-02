
# 🛰️ BIMIP — Binary Interface for Messaging and Internet Protocol

**BIMIP** is a scalable real-time communication protocol built for developers who need reliable, low-latency connectivity across devices, users, and systems.

It is designed for **massive-scale chat**, **awareness**, **streaming**, and **real-time collaboration** applications.  
BIMIP combines **binary efficiency** with **distributed architecture** powered by the BEAM (Erlang/Elixir).

---

## 🚀 Vision

> “To create a universal, binary-based communication interface that connects millions of devices efficiently and securely.”

BIMIP (Binary Interface for Messaging and Internet Protocol) provides an open framework for building **high-performance message brokers** and **device-aware systems** that can run globally distributed.

---

## 🧩 Core Components

| Component | Description |
|------------|-------------|
| **BimipServer** | Core router managing sessions, routing, and awareness events |
| **BimipSignal** | Handles **BimipClient** connections, signaling, and message serialization/deserialization |
| **Orchestrator** | Epokhai session mannager (ESM) Manages and cordinate device communication |
| **Registry** | Tracks connected JIDs and device IDs using ETS |
| **Storage** | Persists message logs, acknowledgments, and device state |

---

## ⚙️ Example Message

```elixir
%BimipServer.AwarenessVisibilityRes{
  id: "1",
  eid: "a@domain.com",
  device_id: "aaaaa1",
  type: 1,
  timestamp: 1761831141324,
  status: 0,
  message: "Visibility updated successfully",
  display_name: "Kunle Tomitope"
}
````

---

## 🔌 Features

* **Binary protocol** for compact payloads
* **Device awareness** (multi-device presence tracking)
* **Scalable routing** using ETS + GenServers
* **Cluster-ready** (node interconnection via RPC or gRPC)
* **Extensible message schema** for offers, candidates, and custom events

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

Perfect 👏 — that’s a **beautifully structured protocol definition** for Bimip.
You now have a complete, modular **stanza documentation** that defines all communication units (messages, signals, awareness, etc.) for your protocol.

Below is a **formal documentation draft** you can include in your repository (`docs/bimip_stanza_spec.md` or `docs/protocol.md`).
It explains purpose, fields, and usage for every stanza — like an internal RFC or developer guide.

---

# 🧩 **Bimip Protocol Stanza Specification**

**Version:** 1.0
**Syntax:** [Protocol Buffers v3](https://developers.google.com/protocol-buffers/docs/proto3)
**Package:** `bimip`

---

## 💡 Overview

**Bimip** (Binary Interface for Messaging and Internet Protocol) defines a universal, signal-based messaging protocol for secure, real-time communication across multiple devices, nodes, and services.

Each stanza below represents a typed message entity for communication between clients, servers, and distributed nodes within the Bimip ecosystem.

---

## 🧱 Base Stanzas

### 🔹 `Identity`

Represents a unique endpoint (user, device, or system node).

| Field                    | Type              | Description                                 |
| ------------------------ | ----------------- | ------------------------------------------- |
| `eid`                    | string            | Global identifier (like JID or account ID)  |
| `connection_resource_id` | string (optional) | Unique session or device ID                 |
| `node`                   | string            | Logical node or cluster handling the entity |

---

### 🔹 `Media`

Metadata for any media attachment associated with a message.

| Field       | Type   | Description                               |
| ----------- | ------ | ----------------------------------------- |
| `type`      | string | `"image"`, `"video"`, `"audio"`, `"file"` |
| `url`       | string | Public or signed URL to resource          |
| `thumbnail` | string | Optional thumbnail URL                    |
| `size`      | int64  | File size in bytes                        |

---

### 🔹 `Signal`

Used for client/server status signaling, acknowledgments, or lifecycle control.

| Field          | Type     | Description                                                                   |
| -------------- | -------- | ----------------------------------------------------------------------------- |
| `id`           | string   | Unique signal ID                                                              |
| `from`         | Identity | Sender identity                                                               |
| `to`           | Identity | Recipient identity                                                            |
| `type`         | int32    | `1=REQUEST`, `2=RESPONSE`, `3=ERROR`                                          |
| `status`       | int32    | `1=TYPING`, `2=RECORDING`, `3=FORWARDED`, `4=DELIVERED`, `5=READ`, `6=RESUME` |
| `timestamp`    | int64    | Epoch milliseconds                                                            |
| `monotonic_id` | int64    | Incremental queue index (per user)                                            |

---

### 🔹 `Payload`

Flexible data structure for arbitrary content.

| Field   | Type                | Description             |
| ------- | ------------------- | ----------------------- |
| `data`  | map<string, string> | Dynamic key-value pairs |
| `media` | repeated `Media`    | Optional attached media |

---

### 🔹 `Metadata`

Cryptographic or security metadata.

| Field       | Type   | Description                                       |
| ----------- | ------ | ------------------------------------------------- |
| `encrypted` | string | Base64 encoded encrypted content                  |
| `signature` | string | Base64 encoded signature for message verification |

---

### 🔹 `Ack`

Acknowledgment status container.

| Field    | Type           | Description                                                             |
| -------- | -------------- | ----------------------------------------------------------------------- |
| `status` | repeated int32 | `9=DELIVERED`, `10=READ`, `11=FORWARDED`, `12=SENT`, `13=PLAYED/VIEWED` |

---

## 🌐 Awareness Stanzas

### 🔹 `Awareness`

Tracks real-time presence, status, and optional location.

| Field                   | Type     | Description                                     |
| ----------------------- | -------- | ----------------------------------------------- |
| `id`                    | string   | Unique awareness ID                             |
| `from`, `to`            | Identity | Sender/receiver                                 |
| `type`                  | int32    | Awareness type (e.g., `1=presence`, `2=status`) |
| `status`                | int32    | Current presence state                          |
| `location_sharing`      | int32    | Whether location is shared (`1=on`, `0=off`)    |
| `latitude`, `longitude` | double   | Optional geo coordinates                        |
| `ttl`                   | int32    | Duration before expiry (seconds)                |
| `details`               | string   | Optional info                                   |
| `timestamp`             | int64    | Epoch time                                      |
| `visibility`            | int32    | `1=public`, `2=contacts`, `3=private`           |

---

### 🔹 `AwarenessContact`

Manages relationship tracking (e.g., “add friend”, “follow”, etc.)

| Field          | Type     | Description               |
| -------------- | -------- | ------------------------- |
| `from`, `to`   | Identity | Participants              |
| `tracking_id`  | string   | Correlation ID            |
| `relationship` | int32    | Relationship type         |
| `action`       | int32    | Action taken (add/remove) |
| `timestamp`    | int64    | Epoch time                |
| `details`      | string   | Description or metadata   |

---

### 🔹 `AwarenessVisibility`

Handles visibility scope updates.

| Field       | Type     | Description        |
| ----------- | -------- | ------------------ |
| `id`        | string   | Request ID         |
| `from`      | Identity | Sender             |
| `type`      | int32    | Visibility type    |
| `timestamp` | int64    | Epoch time         |
| `details`   | string   | Context or comment |

---

## 💬 Messaging Stanzas

### 🔹 `Message`

Primary chat or communication message.

| Field        | Type     | Description                              |
| ------------ | -------- | ---------------------------------------- |
| `id`         | string   | Message ID                               |
| `from`, `to` | Identity | Sender & receiver                        |
| `type`       | string   | Message content type (text, media, etc.) |
| `timestamp`  | int64    | Sent time                                |
| `payload`    | Payload  | Message content                          |
| `ack`        | Ack      | Acknowledgment                           |
| `metadata`   | Metadata | Security metadata                        |

---

### 🔹 `PushNotification`

Server-triggered push messages (non-chat).

| Field        | Type     | Description                   |
| ------------ | -------- | ----------------------------- |
| `id`         | string   | Notification ID               |
| `from`, `to` | Identity | Sender & receiver             |
| `type`       | string   | Notification category         |
| `timestamp`  | int64    | Epoch time                    |
| `payload`    | Payload  | Notification content          |
| `ack`        | Ack      | Optional delivery ACK         |
| `metadata`   | Metadata | Encryption/signature metadata |

---

## 🚦 Control Stanzas

### 🔹 `ErrorMessage`

Reports errors during routing or stanza processing.

| Field          | Type   | Description                         |
| -------------- | ------ | ----------------------------------- |
| `code`         | int32  | Error code                          |
| `error_origin` | int32  | `1=client`, `2=server`, `3=network` |
| `details`      | string | Error description                   |
| `timestamp`    | int64  | When it occurred                    |

---

### 🔹 `PingPong`

Heartbeat or keep-alive stanza for maintaining connections.

| Field       | Type     | Description        |
| ----------- | -------- | ------------------ |
| `id`        | string   | Ping ID            |
| `from`      | Identity | Pinger             |
| `type`      | int32    | `1=ping`, `2=pong` |
| `timestamp` | int64    | Epoch time         |
| `details`   | string   | Optional comment   |

---

### 🔹 `TokenAuthority`

JWT or token exchange mechanism for authentication.

| Field       | Type     | Description         |
| ----------- | -------- | ------------------- |
| `to`        | Identity | Target              |
| `token`     | string   | JWT or token string |
| `type`      | int32    | Token type          |
| `task`      | int32    | Operation type      |
| `timestamp` | int64    | Time of issue       |
| `details`   | string   | Optional message    |

---

### 🔹 `Logout`

Graceful logout or session termination notice.

| Field       | Type     | Description    |
| ----------- | -------- | -------------- |
| `to`        | Identity | Target session |
| `type`      | int32    | Logout type    |
| `status`    | int32    | Logout status  |
| `timestamp` | int64    | Epoch time     |
| `details`   | string   | Optional text  |

---

## 📍 Location & Awareness Streams

### 🔹 `LocationStream`

Ephemeral, non-persistent real-time location updates.

| Field                               | Type     | Description                     |
| ----------------------------------- | -------- | ------------------------------- |
| `id`                                | string   | Unique ID                       |
| `from`, `to`                        | Identity | Sender & receiver               |
| `latitude`, `longitude`, `altitude` | double   | Coordinates (altitude optional) |
| `timestamp`                         | int64    | Epoch time in ms                |

💡 Used for **awareness and live tracking**, not stored in persistence.

---

## 🧠 Advanced Containers

### 🔹 `Body`

A batched stanza body (e.g., awareness updates).

| Field            | Type               | Description       |
| ---------------- | ------------------ | ----------------- |
| `route`          | int64              | Routing key or ID |
| `awareness_list` | repeated Awareness | Awareness updates |
| `timestamp`      | int64              | Epoch time        |

---

### 🔹 `MessageScheme`

Universal envelope for all possible stanza types.

| Field     | Type  | Description                                                                  |
| --------- | ----- | ---------------------------------------------------------------------------- |
| `route`   | int64 | Routing or message path ID                                                   |
| `payload` | oneof | One of: Awareness, PingPong, Message, PushNotification, LocationStream, etc. |

---

## 📘 Notes

* All timestamps use **Unix epoch (milliseconds)**.
* Every stanza can include `from` and `to` fields for routing.
* `MessageScheme` is the **root envelope** for transmission over WebSocket, gRPC, or internal queues.
* Non-persistent stanzas (e.g., `LocationStream`, `PingPong`, `AwarenessVisibility`) are held only in memory.

---

Would you like me to format this as a **GitHub-ready Markdown file** (`bimip_stanza_spec.md`) with headers, code blocks, and a short intro paragraph referencing your GitHub project link (so it’s publishable in your Bimip repo)?


---

## 📄 License

Released under the [MIT License](./LICENSE)

---

## 👨‍💻 Author

**Paul Aigokhai Olukayode**
Full Stack Software Engineer | Architect of BIMIP
📧 [paul.olukayode.pro@gmail.com](mailto:paul.olukayode.pro@gmail.com)
🌐 [GitHub](https://github.com/olukayodepaul) | [LinkedIn](https://linkedin.com/in/paulaigbokhaiolukayode)


---

Next step, we’ll start the **docs** folder:
### → Step 2: `docs/1-overview.md`

Would you like me to create that file next (the one explaining **what BIMIP is, its philosophy, and how it differs from protocols like HTTP, WebSocket, or MQTT**) — or do you want to finalize the README first with an image/logo placeholder?
```
