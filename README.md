# Wireshark Network Traffic Analysis

## Project Overview

This project documents a hands-on network traffic investigation
performed using Wireshark and TShark on Fedora Linux.

The objective was to capture and analyze network traffic,
identify common network protocols, examine DNS activity,
and investigate unusual connection behavior.

A 19.66-second packet capture containing 466 packets was
analyzed to better understand real-world network communication.

## Lab Environment

- Operating System: Fedora Linux
- Hardware: Lenovo ThinkPad T14 Gen 1 (AMD)
- Tools: Wireshark and TShark
- Capture Interface: wlp3s0 (Wi-Fi)
- Capture Format: PCAPNG
- Total Packets: 466
- Capture Duration: 19.66 seconds

## Investigation Methodology

The investigation followed a structured network traffic
analysis workflow.

### 1. Capture Verification

Used TShark and Capinfos to inspect the packet capture
and verify its integrity and metadata.

- Confirmed 466 captured packets
- Verified a capture duration of 19.66 seconds
- Identified the Wi-Fi capture interface
- Generated a SHA256 hash for file integrity verification

### 2. Protocol Analysis

Used TShark protocol hierarchy statistics to identify
the network protocols present in the capture.

Observed protocols included:

- IPv4 and IPv6
- TCP and UDP
- DNS and mDNS
- TLS and QUIC
- ICMP and ICMPv6
- ARP

### 3. DNS Analysis

Extracted DNS query names to investigate which domains
the workstation attempted to resolve.

Observed DNS queries associated with:

- Google services
- Mozilla and Firefox infrastructure
- Content delivery networks (CDNs)
- Common websites and online services

DNS queries were treated as evidence of name resolution
attempts, not proof of successful connections.

### 4. IP Conversation Analysis

Examined IPv4 conversations to identify communicating
hosts, packet counts, transferred bytes, and connection
durations.

Selected a connection involving a Mozilla telemetry
endpoint for further investigation.

### 5. Encrypted Traffic Investigation

Inspected QUIC and TCP/TLS packets communicating with
the same external endpoint.

Examined:

- Source and destination ports
- TCP connection establishment
- TLS Server Name Indication (SNI)
- Packet timestamps
- ICMP error messages
- Encapsulated packet headers

The investigation focused on identifying observable
network behavior without assuming malicious activity.

## Investigation Findings

### Finding 1: QUIC and TCP/TLS Communication

During the investigation, encrypted traffic was observed
between the local workstation and an external server.

TLS Server Name Indication (SNI) identified the service as:

- Domain: incoming.telemetry.mozilla.org
- Destination IP: 34.54.185.247
- QUIC: UDP port 443
- TCP/TLS: TCP port 443

The workstation initiated QUIC communication before
establishing a separate TCP/TLS connection to the
same destination.

The TCP/TLS connection exchanged encrypted data.

### Finding 2: ICMP Port Unreachable Errors

Seven ICMP Destination Unreachable (Type 3, Code 3)
messages were associated with the Mozilla endpoint.

Packet inspection confirmed that the local workstation
generated these errors in response to incoming UDP
packets from the remote server.

The ICMP messages contained embedded IP and UDP
headers identifying the original QUIC traffic.

### Finding 3: Possible Transport Transition

Packet timestamps showed the following sequence:

- 17.135s: Initial QUIC packets transmitted
- 17.170s: Remote server acknowledged QUIC traffic
- 17.183s: TCP connection establishment began
- 17.285s: First ICMP Port Unreachable error
- 19.457s: Additional QUIC traffic and ICMP error

This behavior is consistent with a possible transition
from QUIC to TCP/TLS.

However, the capture does not establish why the
application stopped accepting incoming QUIC traffic
or prove that an intentional fallback occurred.

### Security Assessment

No malicious activity was established from the
observed traffic.

The findings demonstrate network protocol behavior
and connection troubleshooting rather than a
confirmed security incident.

Additional endpoint logs and longer packet captures
would be needed to determine the root cause.

## Tools and Commands

The following tools were used during the investigation:

- Wireshark: Packet capture and visual inspection
- TShark: Command-line packet analysis
- Capinfos: Capture metadata verification
- SHA256: File integrity verification

Example analysis commands:

```bash
# Display capture metadata
capinfos first-capture.pcapng

# Analyze protocol distribution
tshark -r first-capture.pcapng -q -z io,phs

# Examine IPv4 conversations
tshark -r first-capture.pcapng -q -z conv,ip

# Identify DNS queries
tshark -r first-capture.pcapng \
  -Y "dns.flags.response == 0 && dns.qry.name" \
  -T fields -e dns.qry.name | sort -u

# Inspect ICMP Port Unreachable messages
tshark -r first-capture.pcapng \
  -Y "icmp.type == 3 && icmp.code == 3"


```

## Privacy and Limitations

The original packet capture is not publicly distributed
because it may contain private network information,
device identifiers, and browsing-related metadata.

The investigation is limited to a 19.66-second capture.
Encrypted application payloads were not decrypted.

The findings do not establish malicious activity
or prove the reason for the observed transport behavior.

## Skills Demonstrated

- Network packet capture and analysis
- DNS investigation
- TCP/IP protocol analysis
- QUIC and TLS traffic examination
- ICMP error interpretation
- Network troubleshooting
- Evidence-based technical documentation
- Security investigation reporting
