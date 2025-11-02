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
| **BimipSignal** | Handles WebSocket signaling and message serialization |
| **Orchestrator** | Manages distributed node coordination and routing between nodes |
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
