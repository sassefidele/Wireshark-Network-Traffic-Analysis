# Network Traffic Investigation Report

**Case:** NT-001 — Baseline Network Protocol Analysis  
**Date:** 2026-11-09 (capture screenshots)  
**Classification:** Benign traffic analysis / educational lab  
**Analyst environment:** macOS, Wireshark 4.6.9

## Scope and method
Two supplied PCAPNG files were inspected for file integrity and packet counts. Wireshark screenshots supplied during the lab document the specific protocol fields below. Findings are confined to the packets identified; no broader threat-hunting conclusion is implied.

### Evidence inventory
| Artifact | Packets in file | Purpose |
|---|---:|---|
| `SOC-Lab-DNS-Analysis-01.pcapng` | 6,213 | DNS, TCP, TLS |
| `ICMP-Analysis.pcapng` | 1,492 | ICMP test |

## Finding 1 — DNS resolution
- Query frame **519**, response frame **520**; transaction ID `0x0bc2`.
- Client `172.20.10.2` → resolver `172.20.10.1`; response reverses direction.
- Query `rec.safesuggestapps.com`, record type **A**.
- Response: `104.18.1.141` and `104.18.0.141`; no-error status.
- Observed response time **41.704 ms**.
- Assessment: successful name resolution; no suspicious behavior established by this transaction.

## Finding 2 — TCP establishment
- TCP stream **50**, client `172.20.10.2:49843`, server `15.236.221.148:443`.
- Frame **3662** SYN (also ECE/CWR ECN negotiation), time 24.572869.
- Frame **4250** SYN/ACK, time 24.715139.
- Frame **4251** ACK, time 24.715372.
- Relative sequence/acknowledgment values: SYN `Seq=0`; SYN/ACK `Seq=0, Ack=1`; ACK `Seq=1, Ack=1`.
- SYN-to-final-ACK elapsed time **142.503 ms**. This is not the application response time.
- Assessment: three-way handshake completed.

## Finding 3 — TLS session
- TCP stream **50**, same endpoint pair as above.
- Client Hello frame **4253** shows SNI `eu-west-3-latency.thmcell.click`.
- Server Hello frame **4440**: negotiated **TLS 1.2** (`0x0303`).
- Cipher suite **`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`** (`0xC02F`).
- Compression: null. Server certificate and encrypted application-data packets are visible.
- Assessment: encrypted application exchange observed. A cipher suite alone does not validate server trust or establish benign intent. No certificate-chain validation was performed in this report.

## Finding 4 — ICMP reachability
- Client `172.20.10.2`, destination `1.1.1.1`.
- Request/reply frame pairs: **642/646**, **660/663**, **680/685**, **729/737**.
- Four requests and four replies; **0% observed packet loss** for this small sample.
- RTT estimates from visible capture timestamps: **36.16, 35.24, 32.71, 39.91 ms**, average approximately **36.0 ms**.
- Assessment: ICMP Echo connectivity succeeded during the test; this does not establish TCP/HTTPS reachability.

## Overall assessment
The selected DNS, TCP, TLS, and ICMP packets demonstrate normal protocol behavior. There is **insufficient evidence to label any observed destination malicious**. A full security investigation would require endpoint context, process attribution, domain reputation, certificate validation, historical baselines, and additional telemetry.

## Limitations
- Captures reflect ordinary workstation activity, not a controlled malicious-traffic scenario.
- SNI and DNS domain in this report are different and should not be assumed to refer to the same session.
- TLS payload was not decrypted; application contents are unknown.
- Packet timestamps are relative to each capture and do not independently establish wall-clock time.
- PCAPNG files contain more traffic than the selected examples; public release requires privacy review.

## Suggested follow-up
Add a controlled detection case (e.g., known benign unusual-port traffic or a labeled training PCAP), write an analyst hypothesis, document validation and false-positive checks, and map to a relevant detection use case before presenting the project as threat detection.
