# ðŸ“¡ Packet Analyzer â€” Educational Deep Packet Inspection (DPI) Engine

[![C++17](https://img.shields.io/badge/C%2B%2B-17-blue.svg)](https://en.wikipedia.org/wiki/C%2B%2B17)
[![CMake](https://img.shields.io/badge/Build-CMake-green.svg)](https://cmake.org/)
[![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux-lightgrey.svg)](https://github.com/rahuldhiman18/Packet-Analyzer)
[![Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen.svg)](https://github.com/rahuldhiman18/Packet-Analyzer)

> A beginner-friendly, **zero-dependency** Deep Packet Inspection system written in modern C++17.  
> It reads real network capture files (`.pcap`), inspects traffic at the packet level, identifies applications even inside encrypted HTTPS (via TLS SNI), applies blocking rules, and writes out a filtered capture â€” all from scratch.

---

## ðŸŽ¯ What Is This Project? (Simple Explanation)

Imagine the internet as a giant postal system. Every time you visit a website, stream a video, or send a message, your computer ships thousands of tiny envelopes called **packets** across the network.

A **Packet Analyzer** (or DPI engine) is like a smart mail-sorting facility. It:

1. **Opens each envelope** to read the address label (source/destination IP and port).
2. **Peeks inside** to see what kind of letter it is (web traffic, video, chat, etc.).
3. **Recognizes the sender** even if the letter is sealed (for HTTPS, it reads the "SNI" label on the outside of the envelope).
4. **Decides whether to deliver or block** the letter based on rules you set.
5. **Keeps a record** of every conversation so you can see who talked to whom.

This project is a **learning-oriented** version of that facility. It is not a replacement for Wireshark or commercial firewalls â€” it is designed to teach you how network inspection, flow tracking, and multi-threaded packet processing work under the hood.

---
## âœ¨ Key Features

| Feature | What It Does |
|---------|--------------|
| ðŸ“¦ **PCAP File Reader** | Parses standard `.pcap` capture files (Wireshark/tcpdump format) including endianness handling. |
| ðŸ” **Layered Packet Parser** | Manually walks Ethernet â†’ IPv4 â†’ TCP/UDP headers to extract ports, flags, and payload. |
| ðŸŒ **TLS SNI Extraction** | Reads the domain name hidden inside HTTPS `Client Hello` packets (e.g., `youtube.com`). |
| ðŸ“Š **Flow Tracking** | Groups packets into conversations using the **Five-Tuple** (source/destination IP, ports, protocol). |
| ðŸš« **Rule-Based Blocking** | Block traffic by IP address, application, domain (with wildcards), or port. |
| âš¡ **Multi-Threaded Pipeline** | Reader â†’ Load Balancers â†’ Fast Path Workers â†’ Writer (producerâ€“consumer pattern). |
| ðŸ“ˆ **Statistics & Reports** | Live counters, application breakdown, blocked/forwarded totals, and connection reports. |
| ðŸ§¹ **Zero Dependencies** | Pure C++17 Standard Library â€” no Boost, no libpcap, no Wireshark SDK. |

---

## ðŸ—ï¸ Architecture

### High-Level Data Flow

```text
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚   INPUT     â”‚      â”‚  PCAP READER â”‚      â”‚  PACKET PARSER  â”‚      â”‚   SNI EXT.   â”‚
â”‚  (.pcap)    â”‚â”€â”€â”€â”€â”€â–¶â”‚  (L2â†’L4)     â”‚â”€â”€â”€â”€â”€â–¶â”‚  (Ethernet/IP/  â”‚â”€â”€â”€â”€â”€â–¶â”‚  (HTTPS      â”‚
â”‚             â”‚      â”‚              â”‚      â”‚   TCP/UDP)      â”‚      â”‚   domain)    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜
                                                                           â”‚
                                                                           â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                         RULE ENGINE (Block by IP / App / Domain / Port)         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                                                           â”‚
                                                                           â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚   OUTPUT    â”‚â—€â”€â”€â”€â”€â”€â”‚  CONNECTION  â”‚â—€â”€â”€â”€â”€â”€â”‚   FLOW TRACKER  â”‚â—€â”€â”€â”€â”€â”€â”‚   RESULT     â”‚
â”‚  (.pcap)    â”‚      â”‚   WRITER     â”‚      â”‚   (Five-Tuple)  â”‚      â”‚   (Allow/    â”‚
â”‚             â”‚      â”‚              â”‚      â”‚                 â”‚      â”‚   Block)     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Multi-Threaded Pipeline (Full DPI Engine)

The production-grade path uses a **producerâ€“consumer** design so packets are processed in parallel without corrupting shared state:

```mermaid
flowchart LR
    subgraph Reader["ðŸ“¥ Reader Thread"]
        R[PCAP Reader<br/>reads packets from file]
    end

    subgraph LB["âš–ï¸ Load Balancers<br/>(2 threads)"]
        LB0[LB0]
        LB1[LB1]
    end

    subgraph FP["ðŸ”§ Fast Path Processors<br/>(4 worker threads)"]
        FP0[FP0<br/>Flow Tracker + DPI + Rules]
        FP1[FP1]
        FP2[FP2]
        FP3[FP3]
    end

    subgraph Writer["ðŸ“¤ Output Writer Thread"]
        W[Writes allowed packets<br/>to output.pcap]
    end

    R -->|five tuple hash| LB0
    R -->|five tuple hash| LB1
    LB0 -->|flow hash| FP0
    LB0 -->|flow hash| FP1
    LB1 -->|flow hash| FP2
    LB1 -->|flow hash| FP3
    FP0 --> W
    FP1 --> W
    FP2 --> W
    FP3 --> W

    style Reader fill:#e1f5fe,stroke:#01579b
    style LB fill:#fff3e0,stroke:#e65100
    style FP fill:#e8f5e9,stroke:#1b5e20
    style Writer fill:#f3e5f5,stroke:#4a148c
```

**Why this design?**
- **Consistent hashing:** Packets from the same conversation always land on the same worker thread. That means each thread tracks its own flows independently â€” no locks needed on the hot path.
- **Thread-safe queues:** Bounded queues with condition variables move packets between stages safely.
- **Lock-free stats:** Counters use `std::atomic` so reporting never blocks packet processing.

---
## ðŸ”¬ How It Works (Step by Step)

### Step 1 â€” Read the Capture File
The program opens a `.pcap` file (the standard format used by Wireshark). It reads the **global header** (magic number, version, link type) and then streams each packet one by one.

### Step 2 â€” Parse Network Layers
Every packet is peeled like an onion:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Ethernet Header (14 bytes)   â†’  MAC addresses, EtherType   â”‚
â”‚  IPv4 Header (20â€“60 bytes)    â†’  Source IP, Dest IP, Proto  â”‚
â”‚  TCP / UDP Header (8â€“60 bytes)â†’  Source Port, Dest Port     â”‚
â”‚  Application Payload          â†’  Actual data (TLS, HTTPâ€¦)   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Step 3 â€” Identify the Application (SNI Extraction)
HTTPS encrypts the contents, but the **Server Name Indication (SNI)** is sent in plaintext during the TLS handshake. The engine parses the TLS `Client Hello` and extracts the domain:

```
Encrypted HTTPS traffic  â”€â”€â–º  Client Hello  â”€â”€â–º  SNI: "www.youtube.com"
```

This is the foundation of nearly all real-world DPI and parental-control systems.

### Step 4 â€” Track the Flow (Five-Tuple)
Each connection is uniquely identified by:

| Field | Example |
|-------|---------|
| Source IP | `192.168.1.100` |
| Destination IP | `142.250.185.206` |
| Source Port | `54552` |
| Destination Port | `443` |
| Protocol | `TCP` |

All packets sharing these five values belong to the same conversation (flow).

### Step 5 â€” Apply Blocking Rules
You can block traffic by:
- **IP address** (e.g., block a specific user)
- **Application** (e.g., block YouTube)
- **Domain** (e.g., block `*.facebook.com` with wildcards)
- **Port** (e.g., block port 8080)

### Step 6 â€” Write the Output
Allowed packets are written to a new `.pcap` file that you can open in Wireshark to verify the results.

---
## ðŸ“ Project Structure

```text
Packet_Analyzer/
â”œâ”€â”€ README.md                    # This file
â”œâ”€â”€ CMakeLists.txt               # CMake build configuration
â”œâ”€â”€ docs/WINDOWS_SETUP.md        # Detailed Windows setup guide
â”‚
â”œâ”€â”€ include/                     # Header files
â”‚   â”œâ”€â”€ types.h                  # FiveTuple, Connection, AppType, DPIStats
â”‚   â”œâ”€â”€ platform.h               # Portable endian (big/little) conversion
â”‚   â”œâ”€â”€ pcap_reader.h            # PCAP file reader
â”‚   â”œâ”€â”€ packet_parser.h          # Ethernet/IPv4/TCP/UDP parser
â”‚   â”œâ”€â”€ sni_extractor.h          # TLS SNI / HTTP / DNS / QUIC extraction
â”‚   â”œâ”€â”€ connection_tracker.h     # Per-flow state machine
â”‚   â”œâ”€â”€ rule_manager.h           # Blocking rules (IP/App/Domain/Port)
â”‚   â”œâ”€â”€ fast_path.h              # Multi-threaded worker + manager
â”‚   â”œâ”€â”€ load_balancer.h          # Load balancer + consistent hashing
â”‚   â”œâ”€â”€ thread_safe_queue.h      # Thread-safe producer-consumer queue
â”‚   â””â”€â”€ dpi_engine.h             # Engine orchestrator
â”‚
â”œâ”€â”€ src/                         # Source files
â”‚   â”œâ”€â”€ main.cpp                 # Simple single-threaded analyzer (CMake target)
â”‚   â”œâ”€â”€ main_dpi.cpp             # Full multi-threaded DPI engine entry
â”‚   â”œâ”€â”€ main_working.cpp         # Simplified working demo
â”‚   â”œâ”€â”€ main_simple.cpp          # Minimal test version
â”‚   â”œâ”€â”€ pcap_reader.cpp
â”‚   â”œâ”€â”€ packet_parser.cpp
â”‚   â”œâ”€â”€ sni_extractor.cpp
â”‚   â”œâ”€â”€ types.cpp
â”‚   â”œâ”€â”€ connection_tracker.cpp
â”‚   â”œâ”€â”€ rule_manager.cpp
â”‚   â”œâ”€â”€ fast_path.cpp
â”‚   â”œâ”€â”€ load_balancer.cpp
â”‚   â””â”€â”€ dpi_engine.cpp
â”‚
â”œâ”€â”€ examples/                    # Sample captures & generator
â”‚   â”œâ”€â”€ test_dpi.pcap            # Sample input capture
â”‚   â”œâ”€â”€ output.pcap              # Sample output capture
â”‚   â””â”€â”€ generate_test_pcap.py    # Python script to generate test captures
â”‚
â”œâ”€â”€ tests/                       # (Reserved for unit tests)
â”œâ”€â”€ build/                       # CMake build output (generated)
â””â”€â”€ .git/
```

---
## ðŸ› ï¸ Requirements

### Software
| Requirement | Minimum | Notes |
|-------------|---------|-------|
| **C++ Compiler** | GCC 9+ / Clang 10+ / MSVC 2019+ | Must support C++17 |
| **Build System** | CMake 3.16+ | Used for the simple analyzer |
| **Python** | 3.6+ | Only needed to generate test `.pcap` files |
| **Git** | 2.0+ | To clone the repository |

### Verified On This Machine
- **OS:** Windows 10/11 (PowerShell)
- **Compiler:** MinGW-w64 `g++` 14.2.0
- **Build:** CMake 3.31.5 + Ninja
- **Python:** 3.12.10

---

## ðŸš€ How to Run (Windows)

### Option 1 â€” CMake + MinGW (Recommended, Verified âœ…)

This builds the **simple packet analyzer** (`build/packet_analyzer.exe`), which reads a capture and prints a human-readable summary of every packet.

```powershell
# 1. Open PowerShell in the project folder
cd C:\Users\HP\Desktop\Coding\Packet_Analyzer

# 2. Create and configure the build directory
cmake -S . -B build

# 3. Build the project
cmake --build build

# 4. Run the analyzer on the sample capture
.\build\packet_analyzer.exe .\examples\test_dpi.pcap 10
```

**Sample output:**
```
====================================
     Packet Analyzer v1.0
====================================

Opened PCAP file: .\examples\test_dpi.pcap
  Version: 2.4
  Snaplen: 65535 bytes
  Link type: 1 (Ethernet)

========== Packet #1 ==========
Time: 2023-11-15 03:43:20.484823

[Ethernet]
  Source MAC:      00:11:22:33:44:55
  Destination MAC: aa:bb:cc:dd:ee:ff
  EtherType:       0x0800 (IPv4)

[IPv4]
  Source IP:      192.168.1.100
  Destination IP: 142.250.185.206
  Protocol:       TCP
  TTL:            64

[TCP]
  Source Port:      54552
  Destination Port: 443
  Flags:            SYN
...
```

### Option 2 â€” Full Multi-Threaded DPI Engine (Manual Build)

This builds the **flagship DPI engine** with load balancers, fast-path workers, blocking rules, and reports (`dpi_engine.exe`). This target is compiled directly with g++ because it is not yet registered in `CMakeLists.txt`.

```powershell
cd C:\Users\HP\Desktop\Coding\Packet_Analyzer

# Compile the full multi-threaded engine
g++ -std=c++17 -O2 -I include -o dpi_engine.exe `
    src/main_dpi.cpp src/dpi_engine.cpp src/fast_path.cpp src/load_balancer.cpp `
    src/connection_tracker.cpp src/rule_manager.cpp src/sni_extractor.cpp `
    src/types.cpp src/pcap_reader.cpp src/packet_parser.cpp -lpthread

# Run with blocking rules
.\dpi_engine.exe .\examples\test_dpi.pcap .\examples\output.pcap `
    --block-app YouTube --block-ip 192.168.1.50 --lbs 2 --fps 2
```

**Usage:**
```powershell
.\dpi_engine.exe <input.pcap> <output.pcap> [options]

Options:
  --block-ip <ip>       Block traffic from source IP
  --block-app <app>     Block application (YouTube, Facebook, Netflix, etc.)
  --block-domain <dom>  Block domain (supports *.facebook.com wildcards)
  --rules <file>        Load blocking rules from a text file
  --lbs <n>             Number of load balancer threads (default: 2)
  --fps <n>             Fast-path worker threads per LB (default: 2)
  --verbose             Enable verbose output
```

### Option 3 â€” Visual Studio 2022 (GUI)

1. Install **Visual Studio 2022 Community** with the **"Desktop development with C++"** workload.
2. Open VS â†’ **"Open a local folder"** â†’ select the `Packet_Analyzer` folder.
3. VS detects `CMakeLists.txt` automatically.
4. Select **Build â†’ Build All**.
5. Run the executable from the generated `build\` folder.

### Option 4 â€” WSL / Linux

```bash
cmake -S . -B build
cmake --build build
./build/packet_analyzer examples/test_dpi.pcap 10
```

---
## ðŸŽ“ Key Concepts for Interviews

If you are presenting this project in an interview, here are the talking points that demonstrate systems-level understanding:

1. **Five-Tuple Flow Identification**  
   `{src_ip, dst_ip, src_port, dst_port, protocol}` uniquely identifies a network conversation â€” the same concept used by stateful firewalls and NAT.

2. **TLS SNI is the DPI Entry Point**  
   HTTPS encrypts payloads, but the domain name is visible in plaintext during the TLS `Client Hello` handshake. That is how firewalls can block `youtube.com` without decrypting traffic.

3. **Producerâ€“Consumer Pipeline**  
   Reader â†’ Load Balancer â†’ Worker â†’ Writer. Bounded thread-safe queues decouple stages and provide natural backpressure.

4. **Consistent Hashing for Flow Affinity**  
   Hashing the Five-Tuple guarantees all packets of one conversation go to the same worker thread, so connection tracking requires **no mutex on the hot path**.

5. **Low-Level Binary Parsing**  
   Manual extraction of Ethernet/IPv4/TCP headers, handling variable-length headers (IHL, data offset), and converting network byte order (big-endian) to host byte order.

6. **PCAP Format Internals**  
   24-byte global header + 16-byte per-packet header; magic-number endianness detection.

7. **C++17 Concurrency Primitives**  
   `std::thread`, `std::mutex`, `std::condition_variable`, `std::shared_mutex`, `std::atomic`, `std::optional`.

8. **Zero-Dependency Systems Programming**  
   Everything is implemented from scratch using only the C++ Standard Library â€” no libpcap, no Boost, no Wireshark SDK.

> âš ï¸ **Honest note:** This is an *educational simplification*. Production DPI engines (like Suricata, Zeek, or commercial firewalls) also handle TLS 1.3 Encrypted Client Hello (ECH), packet reassembly/defragmentation, kernel bypass (DPDK/AF_XDP), and hardware offloading.

---

## ðŸ”® Limitations & Future Work

- **TLS 1.3 / ECH:** Encrypted Client Hello hides the SNI; modern DPI requires additional techniques.
- **Packet Reassembly:** TCP segments that span multiple packets are not reassembled before inspection.
- **Live Capture:** Currently reads `.pcap` files only; live interface capture would require libpcap/WinPcap.
- **CMake Target:** The full multi-threaded engine (`main_dpi.cpp`) is compiled manually; it is not yet a registered CMake executable target.

---

## ðŸ¤ Contributing

Contributions are welcome! Please:
1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/your-feature`).
3. Commit your changes (`git commit -m "Add: ..."`).
4. Push to the branch (`git push origin feature/your-feature`).
5. Open a Pull Request.

---

## ðŸ“„ License

This project is for educational purposes. See the repository for license details.

---

**Built with â¤ï¸ and pure C++17 â€” no external libraries required.**

