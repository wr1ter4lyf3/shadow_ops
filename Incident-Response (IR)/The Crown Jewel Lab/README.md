# TryHackMe: The Crown Jewel — Network Traffic Analysis

> A defensive investigation of IDS alerts, structured network logs, and packet-capture evidence using Splunk, Bash, `grep`, and Wireshark.

**Spoiler warning:** This write-up discusses the lab's findings and answer values. A compact answer key appears near the end for quick reference.

New to GitHub formatting? See the companion [Markdown refresher](MARKDOWN-REFRESHER.md).

## Overview

This lab presented a simulated network incident involving several suspicious behaviors, including a reverse shell, exploitation attempts against a Jira instance, ARP poisoning, plaintext credential exposure, and DNS-based data exfiltration.

My goal was not only to identify the requested indicators, but also to understand what each artifact showed, how the evidence related across data sources, and where the packet capture supported an inference rather than a certainty.

> [!NOTE]
> All IP addresses, credentials, domains, and attack activity shown here come from an authorized TryHackMe lab environment.

## Skills Practiced

- Investigating an IDS alert in Splunk
- Correlating structured logs with packet-capture evidence
- Filtering text logs with Bash pipelines and `grep`
- Writing Wireshark display filters
- Following TCP conversations
- Recognizing ARP-poisoning behavior
- Inspecting an unencrypted HTTP POST request
- Identifying DNS exfiltration through suspicious subdomains
- Separating confirmed observations from analyst inference

## Tools and Data Sources

| Tool or source | How I used it |
| --- | --- |
| Splunk | Triaged the reverse-shell alert and extracted connection details |
| Bash / `grep` | Narrowed the structured network log to Jira HTTP events |
| Wireshark | Examined TCP, ARP, HTTP, and DNS traffic in the PCAP |
| IDS logs | Identified the initial suspicious outbound connection |
| HTTP logs | Found the non-standard client interacting with Jira |
| PCAP | Validated network behavior at the packet level |

## Investigation

### 1. Triaging the reverse-shell alert in Splunk

The investigation began with the IDS message **Reverse Shell Outbound Connection Detected**. I searched the IDS events in Splunk and inspected the fields attached to the matching alert.

```spl
index=network_logs log_type=ids
```

The event identified:

- Source IP: `10.10.10.100`
- Destination IP: `1.1.1.1`
- Destination port: `8080`
- Transport protocol: `TCP`

![Splunk field showing the reverse-shell source IP](assets/splunk-source-ip.png)

![Splunk field showing destination port 8080](assets/splunk-destination-port.png)

A reverse shell does not have one mandatory port. Port `23` is conventionally associated with Telnet; in this event, the suspected command-and-control connection used TCP port `8080`.

### 2. Confirming the conversation in Wireshark

I pivoted from the alert values into the PCAP and filtered for traffic involving both endpoints and port `8080`:

```wireshark
ip.addr == 10.10.10.100 && ip.addr == 1.1.1.1 && tcp.port == 8080
```

The resulting packets showed the TCP handshake and subsequent conversation between the internal host and external destination.

![Wireshark filter isolating the suspected reverse-shell conversation](assets/reverse-shell-tcp-conversation.png)

To isolate the initiating SYN packet, I could use:

```wireshark
ip.src == 10.10.10.100 &&
ip.dst == 1.1.1.1 &&
tcp.dstport == 8080 &&
tcp.flags.syn == 1 &&
tcp.flags.ack == 0
```

I also used **Follow → TCP Stream** to examine a single conversation without unrelated packets crowding the view. Once Wireshark assigns the conversation a stream number, it can be revisited with:

```wireshark
tcp.stream eq <stream_number>
```

### 3. Finding the non-standard Jira client in the network log

The next task was to identify a non-standard user agent interacting with Jira. The provided `network_logs` file contained structured, key-value events rather than raw packets.

I used two `grep` commands connected by a pipe:

```bash
grep 'host="jira"' ~/Desktop/network_traffic/network_logs \
  | grep 'log_type="http"'
```

This works in two stages:

1. The first `grep` keeps lines associated with `host="jira"`.
2. The pipe (`|`) sends those lines into the second `grep`.
3. The second `grep` keeps only HTTP events.

The suspicious event contained the following indicators:

```text
clientip="1.1.1.1"
method="POST"
uri="/vulnerable_endpoint?cmd=RCE"
status="404"
agent="CVE-202X-EXPLOIT"
c2_port="8080"
```

![Filtered Jira HTTP logs showing the malicious user agent](assets/jira-user-agent.png)

Several user agents appeared in the data, including `curl`, `Python-requests`, browser strings, and a Go HTTP client. Those can be unusual in some environments, but the combination of `CVE-202X-EXPLOIT`, `cmd=RCE`, and `c2_port="8080"` made this event the strongest malicious indicator.

To extract and count the agents instead of manually scanning entire log lines, I could extend the pipeline:

```bash
grep 'host="jira"' ~/Desktop/network_traffic/network_logs \
  | grep 'log_type="http"' \
  | grep -o 'agent="[^"]*"' \
  | sort \
  | uniq -c
```

Here, `grep` is primarily **matching and filtering lines**. The complete pipeline performs a small amount of text processing by filtering, extracting, sorting, and counting the values.

> [!IMPORTANT]
> In this log format, `host="jira"` identifies the system or log source associated with the event. It is not necessarily the same thing as the HTTP `Host:` request header that Wireshark exposes as `http.host`.

### 4. Detecting ARP poisoning

I filtered for ARP traffic and observed a rapid series of unsolicited ARP replies. The same MAC address, `00:0c:29:11:22:33`, repeatedly claimed ownership of both `10.10.10.1` and `10.10.10.150`.

Useful filters included:

```wireshark
arp
```

```wireshark
arp.opcode == 2
```

```wireshark
eth.src == 00:0c:29:11:22:33 && arp
```

![Repeated ARP replies associated with the suspicious MAC address](assets/arp-poisoning.png)

ARP maps IPv4 addresses to MAC addresses on a local network. By sending forged ARP replies, an attacker can convince a victim that the attacker's MAC address belongs to the default gateway. Traffic intended for systems beyond the local network is then delivered to the attacker at Layer 2. If the attacker forwards that traffic onward, the victim's connection may continue working while the attacker occupies a man-in-the-middle position.

High ARP volume alone is not definitive proof of poisoning. The more meaningful evidence here was the repeated conflicting identity claim: one MAC address presenting itself as multiple important IP addresses.

### 5. Identifying exposed credentials in an HTTP POST request

I used the suspicious MAC address and the HTTP request method to narrow the capture:

```wireshark
eth.addr == 00:0c:29:11:22:33 && http.request.method == POST
```

The matching packet was a POST request to `/login.php`. Wireshark decoded the URL-encoded form fields and displayed a username and password directly in the packet details.

![Unencrypted form fields inside an HTTP POST request](assets/plaintext-http-post.png)

The credentials were transmitted in plaintext because the request used unencrypted HTTP. Wireshark displayed the same bytes in two ways:

- The **packet details pane** interpreted them as individual HTML form items.
- The **packet bytes pane** showed hexadecimal byte values alongside their printable ASCII representation.

Although the username and password appeared on separate rows in the dissected form view, the underlying request body followed the familiar structure:

```text
username=<value>&password=<value>
```

### 6. Identifying DNS exfiltration

After locating the suspicious POST request, I used its packet number as a chronological pivot and inspected the packets that followed. This exposed several DNS queries from `10.10.10.200` to `10.10.10.1` containing long, random-looking labels beneath the same base domain.

```wireshark
frame.number > 4891 && dns
```

Once I identified the common domain, I could verify the pattern more directly:

```wireshark
dns.flags.response == 0 && dns.qry.name contains "exfil-domain.xyz"
```

![DNS requests containing encoded-looking subdomains](assets/dns-exfiltration.png)

The repeated query structure was consistent with DNS exfiltration:

```text
<encoded_data_chunk>.exfil-domain.xyz
```

In this technique, software encodes data, places it inside DNS query names, and sends those queries through the normal DNS resolution process. If the attacker controls the authoritative DNS infrastructure for the base domain, their system can log the full query names, remove the domain suffix, reassemble the chunks, and decode the original data. The DNS response may be irrelevant because the query itself carries the payload.

## Evidence vs. Inference

The capture contains several internal systems, so I avoided treating every suspicious packet as one conclusively proven chain.

### Confirmed by the artifacts

- An IDS event identified an outbound TCP connection from `10.10.10.100` to `1.1.1.1:8080`.
- The Jira HTTP logs contained an overt exploit-style user agent and remote-code-execution parameter.
- Repeated ARP replies associated multiple IP addresses with the same suspicious MAC address.
- An HTTP POST request exposed readable form credentials.
- DNS queries contained encoded-looking labels beneath `exfil-domain.xyz`.

### Reasonable interpretation

ARP poisoning can establish a man-in-the-middle position from which plaintext HTTP credentials can be captured. DNS can then serve as a covert channel for moving collected data to attacker-controlled infrastructure. That sequence is technically plausible and consistent with the lab's themes, but each relationship should be demonstrated through shared hosts, MAC addresses, timestamps, or payload content before being stated as fact in a real investigation.

DNS exfiltration also does **not** require ARP poisoning in general. Malware running directly on a compromised endpoint can generate malicious DNS queries without a local man-in-the-middle attack.

## Findings

<details>
<summary><strong>Show the compact lab answer key</strong></summary>

| Finding | Value |
| --- | --- |
| Reverse-shell source | `10.10.10.100` |
| Reverse-shell destination | `1.1.1.1:8080` |
| Transport protocol | `TCP` |
| Jira user agent | `CVE-202X-EXPLOIT` |
| Suspicious ARP MAC | `00:0c:29:11:22:33` |
| HTTP username | `dev_user` |
| HTTP password | `SecretPassword!` |
| Exfiltration domain | `exfil-domain.xyz` |
| Exfiltration protocol | `DNS` |

</details>

## Lessons Learned

This lab refreshed several fundamentals at once:

1. **Start with the alert, then pivot.** Splunk supplied concrete fields that became Wireshark filters.
2. **A reverse shell has no universal port.** The direction and behavior of the connection matter more than memorizing one port number.
3. **Logs and PCAPs answer different questions.** Structured logs summarize events efficiently; packet captures preserve lower-level protocol evidence.
4. **Pipes build a narrowing workflow.** Each command should reduce or transform the previous command's output.
5. **Read the log's schema before choosing a Wireshark field.** A field named `host` in a log may not represent an HTTP host header.
6. **ARP poisoning abuses trust in local address resolution.** The victim sends frames to the attacker's MAC while the IP packet can retain its intended Layer 3 destination.
7. **HTTP form data can be readable on the wire.** HTTPS is what protects the contents in transit.
8. **DNS queries can carry data, not merely request it.** Long, high-entropy, repeated subdomains deserve investigation.
9. **Packet numbers are useful pivots.** When a perfect display filter is not obvious, following the timeline from a known event can reveal the next stage.
10. **Document certainty honestly.** Strong analysis distinguishes direct evidence from a plausible narrative.

## Closing Thoughts

The most valuable part of this lab was reconstructing meaning across several views of the same environment. Splunk helped identify a suspicious connection, Bash made large text logs manageable, and Wireshark exposed the protocol-level details needed to validate and interpret the activity.

The investigation also reinforced that effective packet analysis is not about memorizing every display filter. It is about asking a focused question, narrowing the available evidence, and revising the hypothesis when the results do not match expectations.
