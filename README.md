# FAST-NU CampusLink

A multi-campus communication system for a Computer Networks lab project.

This project implements a central server and campus clients that communicate using TCP (for authentication, routed messages, and file transfer) and UDP (for heartbeat monitoring). It also includes a GTK-based GUI client and a Packet Tracer topology file.

## Suggested Project Name

Primary suggestion: **FAST-NU CampusLink**

Other good options:
- **NU InterCampus Exchange**
- **Campus Relay Network (CRN)**
- **FAST Campus Connect**

## What This Project Does

- Authenticates campus clients using campus-specific credentials.
- Maintains active campus sessions on a central server (Islamabad).
- Routes direct inter-campus department messages.
- Supports file transfer (up to 1 MB) between campuses.
- Tracks client liveness via UDP heartbeats.
- Allows admin broadcast messages to all connected campuses.
- Provides both:
  - CLI client (`client.cpp`)
  - GUI client using GTK (`client_gui.cpp`)

## Project Architecture

- **Server (`server.cpp`)**
  - TCP listener on port `8080`
  - UDP listener on port `8081`
  - Authenticates clients by campus/password map
  - Routes messages and files
  - Monitors heartbeat freshness
  - Runs an admin console for campus list + broadcast

- **Client CLI (`client.cpp`)**
  - Connects/authenticates via TCP
  - Sends heartbeat via UDP every 10 seconds
  - Sends/receives text messages
  - Sends/receives files (hex-encoded payload)
  - Interactive text menu

- **Client GUI (`client_gui.cpp`)**
  - GTK notebook-based interface with tabs:
    - Connection
    - Send Message
    - Send File
    - Messages
  - Receives and displays routed messages/broadcasts
  - Saves received files as `received_<filename>`

## Campuses and Credentials

Configured in server source:

- LAHORE : `NU-LHR-123`
- KARACHI : `NU-KHI-123`
- PESHAWAR : `NU-PWR-123`
- CFD : `NU-CFD-123`
- MULTAN : `NU-MLN-123`

## Message Protocols Used

### Authentication

Client -> Server:

`AUTH:Campus:<CAMPUS>,Pass:<PASSWORD>`

Server -> Client:

- `AUTH:SUCCESS`
- `AUTH:FAILED`

### Direct Text Message

Client -> Server:

`TO:<TARGET_CAMPUS>|DEPT:<TARGET_DEPT>|MSG:<MESSAGE_TEXT>`

Server -> Target Client:

`FROM:<SOURCE_CAMPUS>|DEPT:<TARGET_DEPT>|MSG:<MESSAGE_TEXT>`

### File Transfer

Client -> Server:

`FILE:TO:<TARGET_CAMPUS>|NAME:<FILENAME>|SIZE:<BYTES>|DATA:<HEX_DATA>`

Server -> Target Client:

`FILE:FROM:<SOURCE_CAMPUS>|NAME:<FILENAME>|SIZE:<BYTES>|DATA:<HEX_DATA>`

### Heartbeat

Client -> Server (UDP):

`HEARTBEAT:<CAMPUS>`

### Broadcast

Server -> All Connected Clients:

`BROADCAST:<MESSAGE>`

## Requirements

Recommended environment: **Linux / WSL** (code uses POSIX sockets and `unistd.h`).

- g++ with C++11+ support
- pthread support
- GTK3 development libraries (for GUI client)

Install GTK3 dev package (Ubuntu/Debian):

```bash
sudo apt update
sudo apt install -y build-essential libgtk-3-dev
```

## Build Instructions

From project root:

### Build Server

```bash
g++ -std=c++11 -pthread server.cpp -o server
```

### Build CLI Client

```bash
g++ -std=c++11 -pthread client.cpp -o client
```

### Build GUI Client

```bash
g++ -std=c++11 client_gui.cpp -o client_gui $(pkg-config --cflags --libs gtk+-3.0) -pthread
```

## Run Instructions

### 1. Start Server

```bash
./server
```

### 2. Start One or More CLI Clients

Examples:

```bash
./client LAHORE NU-LHR-123
./client KARACHI NU-KHI-123
./client PESHAWAR NU-PWR-123
```

### 3. (Optional) Start GUI Client

```bash
./client_gui
```

Then connect from the GUI using campus + password.

## Included Test Script

`test_system.sh` launches server and 5 clients in separate `gnome-terminal` windows.

Run:

```bash
chmod +x test_system.sh
./test_system.sh
```

Note: This requires a Linux desktop with `gnome-terminal`.

## Files In This Repository

- `server.cpp` / `server.h` - Central server implementation and definitions
- `client.cpp` / `client.h` - CLI campus client
- `client_gui.cpp` / `client_gui.h` - GTK GUI campus client
- `test_system.sh` - Multi-terminal test launcher
- `CN-LAB-PROJECT.pkt` - Cisco Packet Tracer topology/project file
- `test.txt` - Basic placeholder text file

## Current Limitations

- No encryption/TLS for messages or credentials.
- Credentials are hard-coded in server source.
- Broadcast routine is sent over TCP sockets (despite UDP-oriented naming).
- File payload is hex-encoded text, which increases transfer size.
- No persistent message history/database.
- Primarily Linux-oriented implementation.

## Suggested Future Improvements

- Add TLS and secure credential handling.
- Move credentials and config into external files.
- Add ACK/retry and delivery status for reliability.
- Use binary-safe framing protocol instead of plain delimiters.
- Add message persistence and logs.
- Add CMake/Makefile for easier cross-platform build.

## Academic Note

This project appears to be developed for a CN Lab assignment focused on socket programming, multi-threading, message routing, and basic distributed system design concepts.
