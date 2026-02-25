## DPI Engine – Educational Deep Packet Inspection Project

This project is a learning-oriented DPI system, built to understand how packet parsing, SNI extraction, and basic traffic classification work internally.
It reads packets from a PCAP file, inspects headers and early payload, identifies applications using TLS SNI, and demonstrates simple rule-based blocking.

This project is built entirely using C++17 + standard libraries, without external dependencies.

# Why This Project Exists

As a student learning networking + systems programming, I wanted to understand:

How raw packets are structured

How Wireshark-type tools parse data

How applications can be detected even in HTTPS

How basic blocking/filtering logic is implemented

How flows are tracked

How multi-threading can improve system throughput

This system achieves those goals through a clear, step-by-step design.

> **NOTE:** This project is an educational simplification of real DPI engines.

###  Project Overvie:-

PCAP File (.pcap)
        ↓
 Pcap Reader (reads raw bytes)
        ↓
 Packet Parser (Ethernet → IPv4 → TCP)
        ↓
 SNI Extractor (detects domains like youtube.com)
        ↓
 Rule Engine (block/allow decisions)
        ↓
 Output PCAP (filtered packets)


> **CORE CONCEPT:** A connection is uniquely identified by the Five-Tuple:-

1) PCAP File Reading

You learn how Wireshark saves captured packets and how to parse:

Global header

Per-packet header

Raw packet data

2) Packet Parsing

You manually extract:

Ethernet header

IPv4 header

TCP header

Application payload pointer

This builds extremely strong intuition for network protocols.

3) TLS SNI Extraction
 **IMPORTANT:** Even though HTTPS is encrypted, the domain name (SNI)
 is visible in the Client Hello packet of TLS. This is the foundation 
 of all DPI systems


## This project teaches:

How TLS Client Hello works

How to locate the SNI extension

How to identify websites (youtube.com, facebook.com, etc.)

 4) Simple Flow Tracking

You learn the concept of Five-Tuple:

Source IP, Destination IP,
Source Port, Destination Port,
Protocol

All packets of a connection are grouped together.

5) Basic Blocking Rules

You implement simple rule types:

Block by IP

Block by App

Block by Domain substring

Enough to understand how firewalls/DPI tools work internally.

## Folder Structure:-

PacketAnalyzer/
│
├── src/
│   ├── main.cpp              # Or main_working.cpp
│   ├── pcap_reader.cpp
│   ├── packet_parser.cpp
│   ├── sni_extractor.cpp
│   ├── types.cpp
│
├── include/
│   ├── pcap_reader.h
│   ├── packet_parser.h
│   ├── sni_extractor.h
│   ├── types.h
│
├── examples/
│   ├── test_dpi.pcap
│   ├── output.pcap
│
├── tests/                    # optional
├── docs/                     # optional notes
│
├── CMakeLists.txt
└── README.md

Clear, simple, easy to understand.

## **Architecture Diagram**:-

+------------------+
|   PCAP Reader    |
|  (raw packets)   |
+--------+---------+
         |
         v
+--------+---------+
| Packet Parser    |
| Ethernet / IP /  |
| TCP extraction   |
+--------+---------+
         |
         v
+--------+---------+
| SNI Extractor    |
| TLS ClientHello  |
| domain detection |
+--------+---------+
         |
         v
+--------+---------+
| Rule Engine      |
| Block / Allow    |
+--------+---------+
         |
         v
+------------------+
| Output Writer    |
+------------------+
 
 ## What the Simple Version Does

1 Reads PCAP
2 Parses Ethernet, IPv4, TCP headers
3 Extracts early payload
4 Identifies domains using TLS SNI
5 Classifies basic apps
6 Tracks flows using 5-tuple
7 Applies simple blocking rules
8 Writes filtered packets to output PCAP



# Multi-Threading (Scaled-Down Explanation)

1 A producer thread (reader)
2 A consumer thread (parser + classifier)
3 A separate writer thread

 # Reader Thread → Processing Thread → Writer Thread

This teaches me the basics of:

thread-safe queues

producer-consumer pattern

low-level concurrency


##  Building:-
1) mkdir build
2) cd build
3) cmake ..
4) cmake --build .

# Run:

./packet_analyzer ../examples/test_dpi.pcap 10

## Key Learning Outcomes:-

1) Understanding of Network Layers
2) Hands-on experience with binary parsing
3) Working with real PCAP files
4) Implementing SNI-based classification
5) Flow-based tracking like real firewalls
6) Clean C++ modular design
7) Basic multi-threading concepts
8) Strong systems-level debugging skills
