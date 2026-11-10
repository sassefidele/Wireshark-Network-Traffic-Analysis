# Wireshark Network Traffic Analysis Lab

**Portfolio project #6 | Network security / SOC fundamentals**  
**Tool:** Wireshark 4.6.9 on macOS | **Capture format:** PCAPNG

## Objective
Capture and analyze normal network traffic to demonstrate practical proficiency in DNS, TCP, TLS, and ICMP analysis. This is a **baseline traffic investigation**, not a claim of malware detection or incident response.

## Environment
- Analyst workstation: macOS, Wireshark capture on Wi-Fi (`en0`).
- Observed client IPv4: `172.20.10.2` (from packets; device identity not independently established).
- DNS resolver/gateway observed: `172.20.10.1`.
- Two original packet captures and four annotated-context screenshots are included.

## Investigations and evidence

| Protocol | What was verified | Evidence |
|---|---|---|
| DNS | Query/response ID `0x0bc2`, domain `rec.safesuggestapps.com`, two IPv4 answers | [DNS screenshot](screenshots/01-DNS-Response.png) |
| TCP | Stream 50: SYN → SYN/ACK → ACK, client `172.20.10.2:49843` to `15.236.221.148:443` | [TCP screenshot](screenshots/02-TCP-Handshake.png) |
| TLS | Server Hello selected TLS 1.2, cipher `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` | [TLS screenshot](screenshots/03-TLS-Server-Hello.png) |
| ICMP | Four echo requests and four replies to/from `1.1.1.1` | [ICMP screenshot](screenshots/04-ICMP-Requests-Replies.png) |

## Reproduce in Wireshark
Open the captures in `captures/` and apply these **display filters**:

```wireshark
dns.id == 0x0bc2
```
```wireshark
tcp.stream == 50 && (frame.number == 3662 || frame.number == 4250 || frame.number == 4251)
```
```wireshark
tcp.stream == 50 && tls
```
```wireshark
icmp
```

> Note: Frame numbers and TCP stream IDs refer specifically to the included capture. They will differ in new captures. The TLS flow and DNS transaction are **separate** examples; the DNS query is not evidence that it resolved the TLS endpoint.

## Key findings
- DNS query `rec.safesuggestapps.com` returned `104.18.1.141` and `104.18.0.141` with a normal/no-error response; measured query-response time ~41.7 ms.
- TCP three-way handshake completed for a connection to destination port 443; observed elapsed time from SYN to final ACK ~142.5 ms.
- The inspected TLS Server Hello negotiated TLS 1.2 with ECDHE/RSA/AES-128-GCM/SHA256; subsequent TLS application data was encrypted.
- Four ICMP echo requests to `1.1.1.1` received four corresponding replies in the captured sample (0% observed loss).
- **Security conclusion:** No malicious behavior can be established from these selected packet examples alone.

## Files
- [`captures/SOC-Lab-DNS-Analysis-01.pcapng`](captures/SOC-Lab-DNS-Analysis-01.pcapng)
- [`captures/ICMP-Analysis.pcapng`](captures/ICMP-Analysis.pcapng)
- [`reports/Network-Traffic-Investigation.md`](reports/Network-Traffic-Investigation.md)

## Skills demonstrated
Packet capture, Wireshark display filtering, DNS request/response correlation, TCP flag/sequence analysis, TLS handshake and cipher-suite inspection, ICMP diagnostics, evidence-based reporting.

## Responsible publication
Packet captures can expose browsing metadata, device/network identifiers, and other personal traffic. **Review and sanitize the PCAPNG files before uploading them to a public GitHub repository.** You may publish screenshots and a redacted report first, and keep raw captures private.
