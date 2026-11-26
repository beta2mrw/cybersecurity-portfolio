# Lab 02 — Network Traffic Analysis with Wireshark

## Objective

Capture and analyze live network traffic using Wireshark to understand network protocols, DNS resolution, TCP connection establishment, HTTP communication, TLS encryption, and network conversation statistics.

The lab focuses on developing practical packet analysis skills relevant to network troubleshooting and Security Operations Center (SOC) investigations.

## Environment

- **Operating System:** macOS
- **Capture Interface:** `en0` (Wi-Fi)
- **Tool:** Wireshark
- **Traffic Source:** Browser-generated traffic
- **Websites visited:** `neverssl.com` and `example.com`

## Tasks Performed

### 1. Live Packet Capture

Captured network traffic from the active Wi-Fi interface `en0` while browsing websites.

- Total packets captured: **1,754**
- Dropped packets: **0**

### 2. DNS Analysis

Applied the display filter:

```wireshark
dns
```

Observed DNS queries and responses generated during browsing activity.

Domains observed included:

- `neverssl.com`
- `coolwonderfulgrandplan.neverssl.com`
- `ab.chatgpt.com`

Inspected DNS query names, record types, query packets, and responses.

### 3. TCP Connection Initiation

Applied the display filter:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Observed TCP SYN packets initiating connections to remote endpoints on ports 80 and 443.

TCP SYN retransmissions were also observed. Retransmissions may occur when responses are delayed or lost, or when a connection attempt does not receive a timely response. Retransmissions alone are not evidence of malicious activity.

### 4. TCP Three-Way Handshake

Applied the display filter:

```wireshark
tcp.flags.syn == 1
```

Inspected the TCP connection associated with:

- Client IP: `192.168.100.12`
- Server IP: `34.223.124.45`
- Client port: `56447`
- Server port: `80`

Verified the connection establishment sequence:

1. SYN — Client initiates the connection.
2. SYN-ACK — Server acknowledges and responds.
3. ACK — Client acknowledges the response.

The handshake confirmed successful TCP connection establishment.

### 5. HTTP Traffic Analysis

Applied the display filter:

```wireshark
http
```

Observed the following HTTP exchange:

| Frame | Activity | Result |
|---|---|---|
| 585 | `GET /online` | Initial request |
| 591 | HTTP response | `301 Moved Permanently` |
| 593 | `GET /online/` | Redirect-following request |
| 599 | HTTP response | `200 OK` |
| 610 | `GET /favicon.ico` | Icon request |
| 616 | HTTP response | `200 OK` |

Inspected HTTP headers, including the request method, URI, hostname, HTTP version, User-Agent, connection information, and Referer.

This demonstrated that plain-text HTTP traffic exposes request details and response information to network observers.

### 6. TLS Client Hello Analysis

Applied the display filter:

```wireshark
tls
```

Inspected the TLS Client Hello packet.

Observed:

- Client IP: `192.168.100.12`
- Server IP: `34.223.124.45`
- Destination port: `443`
- TLS stream index: `11`
- SNI hostname: `coolwonderfulgrandplan.neverssl.com`

The Client Hello provided information about the connection's TLS negotiation and the hostname being requested through Server Name Indication (SNI).

### 7. TLS Server Hello Analysis

Applied the display filter:

```wireshark
tls.handshake.type == 2
```

Inspected the Server Hello response from the remote server.

The Server Hello was observed in the same TLS stream (stream index 11), allowing examination of the server's response during TLS negotiation.

### 8. Encrypted TLS Application Data

Applied the display filter:

```wireshark
tls.app_data
```

Inspected Frame 557.

| Field | Observed value |
|---|---|
| Frame | 557 |
| Source IP | `34.223.124.45` |
| Destination IP | `192.168.100.12` |
| Source port | 443 |
| Destination port | 64075 |
| Protocol | TLS 1.3 |
| Packet size | 90 bytes |
| TCP payload length | 24 bytes |
| TLS record length | 19 bytes |
| TLS record type | Application Data (23) |
| TLS stream | 11 |

Wireshark identified the payload as encrypted application data. The underlying HTTP content could not be read from this packet without the appropriate decryption keys.

### 9. TCP Stream Analysis

Used Wireshark's **Follow TCP Stream** functionality to reconstruct the TCP conversation associated with stream 11.

The stream was associated with the HTTPS connection between:

- Client: `192.168.100.12:64075`
- Server: `34.223.124.45:443`

The application content remained encrypted, demonstrating the limitations of inspecting HTTPS traffic without TLS decryption keys.

### 10. TCP Conversation Statistics

Inspected the TCP conversation through:

**Statistics → Conversations → TCP**

Conversation details:

| Metric | Value |
|---|---:|
| Client IP | `192.168.100.12` |
| Client port | `64075` |
| Server IP | `34.223.124.45` |
| Server port | `443` |
| Duration | 5.881979 seconds |
| Total packets | 28 |
| Client → Server packets | 12 |
| Server → Client packets | 16 |
| Total bytes | 9,178 |
| Client → Server bytes | 3,575 |
| Server → Client bytes | 5,603 |
| Client → Server throughput | 4,862 bits/s |
| Server → Client throughput | 7,620 bits/s |

The server transmitted more bytes than the client during the observed conversation, consistent with a web client receiving data from a remote server.

### 11. Protocol Hierarchy Analysis

Inspected the protocol breakdown through:

**Statistics → Protocol Hierarchy**

| Protocol | Packets | % Packets | Bytes | % Bytes |
|---|---:|---:|---:|---:|
| Frame | 28 | 100% | 9,178 | 100% |
| Ethernet | 28 | 100% | 392 | 4.27% |
| IPv4 | 28 | 100% | 560 | 6.10% |
| TCP | 28 | 100% | 892 | 9.72% |
| TLS | 11 | 39.29% | 7,334 | 79.91% |

TLS represented approximately 79.91% of the conversation's captured bytes.

Protocol hierarchy percentages represent encapsulated protocol layers and should not be added together as independent traffic categories.

## Evidence

Screenshots are stored in the `screenshots/` directory.

| Screenshot | Description |
|---|---|
| `01-dns-traffic.png` | DNS query and response analysis |
| `02-tcp-syn-packets.png` | TCP connection initiation and SYN packets |
| `03-tcp-three-way-handshake.png` | TCP handshake sequence |
| `04-http-request-analysis.png` | HTTP request and response inspection |
| `05-tls-client-hello.png` | TLS Client Hello and SNI |
| `06-tls-server-hello.png` | TLS Server Hello |
| `07-tls-encrypted-data.png` | TLS encrypted application data |
| `08-protocol-hierarchy.png` | Protocol hierarchy statistics |

## Findings

- DNS queries and responses were captured and inspected.
- TCP connection attempts were observed using SYN packets.
- SYN retransmissions were observed and considered in context rather than treated as proof of malicious activity.
- A complete TCP three-way handshake was verified.
- HTTP GET requests, a 301 redirect, and 200 OK responses were observed.
- HTTP headers and requested resources were visible in plain text.
- TLS Client Hello and Server Hello messages were inspected.
- SNI exposed the requested hostname in the observed TLS handshake.
- TLS Application Data was identified as encrypted payload.
- TCP stream reconstruction was used to inspect the conversation.
- Conversation statistics showed 28 packets and 9,178 bytes exchanged over approximately 5.88 seconds.
- TLS accounted for approximately 79.91% of the conversation's captured bytes.

## Security Relevance

Network traffic analysis is an important SOC analyst skill. Wireshark can help analysts:

- Investigate suspicious network connections.
- Identify communicating hosts and services.
- Analyze DNS activity.
- Inspect TCP connection behavior.
- Investigate retransmissions and connection failures.
- Examine HTTP requests and responses.
- Identify TLS connections and observable handshake metadata.
- Understand the limitations of inspecting encrypted traffic.
- Correlate packet-level observations with other security logs and telemetry.

Encrypted traffic is not necessarily malicious. Analysts should use context, behavioral evidence, reputation data, and corroborating information before making security judgments.

## Lessons Learned

This lab provided hands-on experience capturing and analyzing network traffic with Wireshark. I learned how to apply display filters, inspect DNS queries and responses, identify TCP SYN packets, verify the TCP three-way handshake, recognize retransmissions, analyze HTTP requests and responses, and inspect TLS handshake messages.

I also learned how to examine encrypted TLS application data, reconstruct TCP streams, review conversation statistics, and use Protocol Hierarchy to understand the composition of network traffic.

The lab reinforced the importance of distinguishing observable network metadata from encrypted application content and of making evidence-based security assessments instead of assuming suspicious activity from isolated indicators.

## Tools Used

- Wireshark
- macOS Terminal
- Web browser
- DNS and TCP display filters
- Wireshark Follow TCP Stream
- Wireshark Conversations
- Wireshark Protocol Hierarchy

## Conclusion

This lab strengthened my foundational network analysis skills by providing practical experience with packet capture, DNS, TCP, HTTP, TLS, and encrypted traffic.

These techniques form an important foundation for SOC investigations, network troubleshooting, and security monitoring.
