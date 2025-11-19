**Lab Date:** Sept 22 → Revisited Nov 19 2025  
**Tooling:** Kali Linux / Nmap  
**Focus:** Port enumeration → attack-surface reduction via service hardening  

This lab directly reinforces Security+ Objectives 2.2 (Threat Vectors & Attack Surfaces), 2.3 (Vulnerability Types), and 2.5 (Mitigation Techniques) by performing practical enumeration of external, guest, and internal network segments using Nmap. The findings map cleanly to real-world hardening recommendations, configuration enforcement, segmentation strategy, and legacy system risk management.

| **Section of your lab**                                          | **Objective 2.2 concept demonstrated**                                                            | **Objective 2.3 concept demonstrated**                                                                       | **Objective 2.5 concept demonstrated**                   | **Mitigation technique**                                                                          |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Discovering outward-facing open ports (border router)**        | Open service ports as a **threat vector**; external **attack surface** exposure                   | **Misconfiguration** (unnecessary external services); **cryptographic vulnerability** (unencrypted services) | **Segmentation / Access control / Hardening techniques** | Close unnecessary ports (21, 22, 25, 3389); restrict admin services; enforce least privilege      |
| **Guest-network scan (OPNsense firewall exposed)**               | **Unsecure network**; management interface reachable from guest VLAN; expanded **attack surface** | **Privilege escalation risk**; **misconfiguration**; **cryptographic vulnerabilities** (HTTP/80 & 8000)      | **Configuration enforcement / Isolation**                | Block firewall GUI from guest VLAN; apply segmentation; enforce HTTPS-only                        |
| **Internal-server scan (Windows Server 2016 EOL)**               | Internal threat vectors; **open service ports enabling lateral movement**                         | **Unsupported/legacy OS**; **service misconfiguration**; unnecessary running services                        | **Decommissioning / Patching / Encryption**              | Replace/upgrade EOL server; restrict services to role-specific ports; enforce encrypted protocols |
| **Numerous internal open ports (SMB, MSRPC, MySQL, IMAP, SMTP)** | Large internal **attack surface**; insider threat vector; pivot opportunities                     | **Misconfiguration vulnerabilities**; excessive services; **unnecessary administrative protocols**           | **Hardening / Least privilege / Segmentation**           | Enable host firewalls; limit allowed ports; apply role-based service minimization                 |
| **Reflection on open-port risk (lab summary)**                   | Recognition of ports as an **attack surface** component                                           | Identifying **vulnerabilities** created by exposure                                                          | **Hardening techniques**                                 | Disable insecure/default services; implement HIPS/HIDS; enforce baseline configurations           |


Commands Used
--

All three major scans used a consistent scan structure:

```bash
nmap [target IP] -F -sS -sV -O -oN [output-file]
```

Only the target IP address and output filename changed as Kali was moved between the Internet subnet, the Guest VLAN, and the Client VLAN (external internet-facing router, internal legacy server Windows Server 2008, guest VLAN gateway). 
This highlights how Bash/Nmap workflows can stay standardized while still adapting to different reconnaissance contexts.

Command List
-

```bash
nmap 203.0.113.1 -F -sS -sV -O -Pn -oN border-scan.nmap
```

<img width="1260" height="427" alt="Screenshot 2025-11-19 054123" src="https://github.com/user-attachments/assets/e9b1c5ca-d155-4f81-bd3a-b1a813712ac0" />

```bash
grep open border-scan.nmap
```

<img width="1252" height="155" alt="Screenshot 2025-11-19 054305" src="https://github.com/user-attachments/assets/f16b017b-8d3a-4a04-b7b7-3168f8a76a93" />



```bash
grep OS border-scan.nmap
```

<img width="1247" height="217" alt="Screenshot 2025-11-19 054504" src="https://github.com/user-attachments/assets/86e6604a-c7ac-4126-8ba2-da356c3b5ca1" />

```bash
dhclient -r && dhclient
ip a s eth0
```

<img width="1263" height="303" alt="Screenshot 2025-11-19 055107" src="https://github.com/user-attachments/assets/c99847a6-d94a-4ffc-af60-a27c3bfb3951" />

```
