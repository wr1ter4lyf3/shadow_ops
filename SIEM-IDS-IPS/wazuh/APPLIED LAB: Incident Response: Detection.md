# Detecting Logon Events with Wazuh

### Objective
This lab focused on the **Detection** phase of the Incident Response cycle  
(Prepare → Detect → Analyze → Contain → Eradicate → Recover → Lessons Learned).  
The goal was to understand how a SIEM detects indicators of compromise (IoCs)  
and how those alerts align with MITRE ATT&CK tactics.

---

### Tool Overview
**Wazuh** is an open-source security platform built on **OSSEC**, designed for log analysis,  
file integrity monitoring, vulnerability detection, intrusion detection, configuration assessment,  
and incident response. It can be integrated with Elastic Stack for visualization  
and deployed on-prem, in the cloud, or hybrid environments.  
In practice, wazuh functions as a **SIEM**, **IDS**, and partial **SOAR** solution.

---

### Lab Setup
- **Environment:** Ubuntu Server VM running wazuh Manager  
- **Agent:** Windows Domain Controller (DC10)  
- **Data Source:** Windows Event Security Logs  
- **Framework Reference:** MITRE ATT&CK for tactic and technique mapping  

---

### Key Activities
- Explored the wazuh web interface to identify and categorize security events.  
- Observed how wazuh correlates Windows log data and assigns **alert levels (0–16)**:  
  - *0–3:* Informational  
  - *4–7:* Low urgency  
  - *8–11:* Medium  
  - *12–15:* High  
  - *16:* Emergency  
- Detected and analyzed simulated adversary behavior, including:  
  - **Pass-the-Hash (PtH)** attacks using Hydra  
  - **Audit log clearing** in Windows Event Viewer    
- Reviewed IoCs (Indicators of Compromise) and observed how Wazuh mapped them to MITRE ATT&CK  
  tactics like *Privilege Escalation*, *Persistence*, and *Defense Evasion*.

---

### Takeaways
- Gained insight into how SIEMs help analysts correlate raw log data into actionable intelligence.  
- Practiced navigating and filtering events to identify false positives vs. genuine threats.  
- Saw how MITRE ATT&CK provides context for adversary techniques within security alerts.  
- Reinforced the importance of understanding alert severity and custom rule creation  
  in real-world SOC environments.

---

### Reflection
This lab gave me a clearer picture of what day-to-day SOC monitoring looks like—  
balancing automation with human analysis. It also tied together core Security+ objectives  
(incident response, log analysis, and threat intelligence) with real-world tooling.
