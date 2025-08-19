# Mini Redis (v07)

A compact, educational Redis-like server implemented in **C++**.  
This version demonstrates core server concepts such as **non-blocking TCP I/O**, an **event loop with `poll()`**, a **length-prefixed binary protocol with pipelining**, incremental parsing, and a simple in-memory key/value store (`get`/`set`/`del`).

---

## 📂 Repository Layout

- `07_server.cpp` — Full server implementation (single file)
- `07_client.cpp` — Minimal client for sending framed commands
- `README.md` — Project documentation

---

## ✨ Features

- Single-threaded, event-driven server using **non-blocking sockets + poll()**
- **Length-prefixed binary protocol** supporting pipelined requests
- Supported commands:
  - `set <key> <value>`
  - `get <key>`
  - `del <key>`
- Per-connection **incoming/outgoing buffers** with incremental parsing
- Clean, readable codebase for **learning and extension**

---

## 📡 Protocol

### Request (binary)

- `4 bytes`: total body length (`uint32_t`, little-endian)
- Body:
  - `4 bytes`: number of strings (`nstr`)
    - For each string:
      - `4 bytes`: length (`uint32_t`)
      - `len` bytes: string data

**Example request for `set k v`:**

[ total_len ] [ nstr ] [ k_len ] [ k ] [ v_len ] [ v ]

text

### Response (binary)

- `4 bytes`: total response length = `4 + payload_size`
- `4 bytes`: status code
  - `0` = OK
  - `1` = ERR
  - `2` = NX (not found)
- Payload: optional data (e.g., value for `GET`)

---

## ⚙️ Build & Run

### Requirements

- Linux / WSL (uses POSIX APIs: `poll.h`, `fcntl`, `unistd`)
- g++ (C++11 or higher)

### Build

g++ -o server 07_server.cpp
g++ -o client 07_client.cpp

text

### Run

./server
./client

text

---

## 🛠 How It Works

### Connection Handling

- Non-blocking listening socket
- Each client assigned a `Conn` object (file descriptor and buffers)

### Event Loop

- `poll()` waits for socket readiness
- Accept new clients, call `handle_read` / `handle_write`

### Parsing & Buffers

- Incoming bytes appended to buffer
- `try_one_request()` ensures full frame, parses via `parse_req()`
- Supports **pipelining** (multiple requests queued)

### Command Execution

- `get`, `set`, `del` on in-memory `std::map<string, string>`
- Response built with `make_response()`

### Non-blocking Writes

- Write as much as possible, adjust flags if data pending

---

## 🚧 Limitations

- **POSIX-only** (use WSL for Windows)
- Single-threaded — limited concurrency
- No persistence (data lost on restart)
- Minimal input validation (guarded by `k_max_msg`)
- Uses `std::map` — not optimized for high throughput

---

## 🚀 Possible Improvements

- Switch from **poll()** → **epoll/kqueue** for better scaling
- Add persistence (AOF or RDB snapshots)
- Support TTLs, lists, sets, sorted sets
- Add connection timeouts & quotas
- Unit & integration testing
- Windows port (Winsock/IOCP)
