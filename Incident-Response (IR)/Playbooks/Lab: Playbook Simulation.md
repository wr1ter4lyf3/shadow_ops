## Playbook Steps (Broad and adaptive)

1. Investigate the high CPU usage and determine the rogue process's name.<br>
2. Terminate the offending process. <br>
3. Hash the file associated with the rogue process. <br>
4. Perform an online malware analysis using the hash value of the suspicious file. <br>
5. Determine the owner of the suspicious file. <br>
6. Archive the suspicious file into a zip container along with a file of its hash value. <br>
7. Copy the zip archive of the suspicious file to a quarantine system. <br>
8. Remove the suspicious file from the affected system(s). <br>
9. Fill out an incident report and submit it to the SOC for review. <br>

## Background on SOAR: 

Security Orchestration, Automation, and Response (SOAR) is a security solution whose purpose is to scan security and threat intelligence data collected from multiple sources within the enterprise and then analyze it using various techniques. A SOAR can also assist with provisioning tasks, such as creating and deleting user accounts, making shares available, or launching VMs from templates. The SOAR will use technologies such as cloud and SDN/SDV APIs, orchestration tools, and cyber threat intelligence (CTI) feeds to integrate the different systems it manages. It will also leverage technologies such as automated malware signature creation and user and entity behavior analytics (UEBA) to detect and identify threats. The automated actions performed by a SOAR are to be documented in runbooks. However, when the SOAR fails to operate properly, security personnel can use a playbook to perform manually the actions that the SOAR would have automated.

When a playbook utilizes a high degree of automation from a SOAR system, it can be referred to as a runbook, though the terms are also widely used interchangeably. A runbook aims to automate as many playbook stages as possible while incorporating clearly defined interaction points for human analysis. These interaction points should present contextual information and guidance needed for an analyst to make a quick, informed decision about the best way to proceed with incident mitigation. For example, a runbook may use integrations for cloud-based email platforms and antimalware solutions. The runbook may take email attachments from user emails and submit them to an online malware detection engine. Suppose such a service identifies the file as malicious. In that case, the SOAR can provide a new custom detection signature to the antimalware software to locate and block any other instances of the malware.

In this lab, I have chosen to address the high CPU usage through command lines

CLI Method: 
<img width="1224" height="632" alt="image" src="https://github.com/user-attachments/assets/f852853e-048d-4200-b90b-4d51f77fa1df" />
`wmic path Win32_PerfFormattedData_PerfProc_Process get Name,PercentProcessorTime.`

Incident response playbooks are an invaluable tool for organizations to quickly and efficiently respond to security incidents. With an incident response playbook, organizations can define the steps they need to take to respond to a security incident, such as the specific roles, processes, and procedures that security staff must follow. Incident response playbooks can also guide communication with stakeholders and the public, as well as how to gather evidence and determine the incident's root cause. Oftentimes, the playbook is just that-a physical book a security professional uses in response to an incident. Using a physical book ensures its availability during a wide-scale incident. In a highly secure environment, it also ensures attackers do not digitally exfiltrate the IR capabilities.

The most effective incident response playbooks are tailored to an organization's specific security needs and provide detailed guidance on responding to various security incidents. For example, a playbook may contain detailed instructions on responding to a ransomware attack or a data breach. Additionally, the playbook should include guidance on taking the necessary steps to contain the incident, such as isolating affected systems and measures to ensure the incident is fully resolved.

Step 3: Hash the suspicious file using the CLI

`cd c:\ && dir /s HeavyLoad.exe.`

This command changes the working or current directory to the root of drive C:\, then searches all sub-folders for the file HeavyLoad.exe.

There should be two results indicating that HeavyLoad.exe is located in both c:\Users and c:\Program Files\JAM Software\HeavyLoad\. For this lab, assume the only result is in c:\Users

`certutil -hashfile "c:\Users\HeavyLoad.exe" SHA256 > c:\Users\HeavyLoad-Hash.txt.` This command will calculate the SHA256 hash of the suspicious executable and save it in an output file.

<img width="1222" height="641" alt="image" src="https://github.com/user-attachments/assets/56a0d9e5-d185-4533-b656-553c8458698e" />

<img width="1217" height="200" alt="image" src="https://github.com/user-attachments/assets/b79ddeef-1d2e-4661-bb86-082c327bbacb" />

## Online Malware Analysis & IDing Owner of Malware

<img width="1884" height="951" alt="image" src="https://github.com/user-attachments/assets/3b32c41b-6ba0-4f7b-ba11-995043853b19" />

A free database meant to help search for malware that's been reported and documented. After confirming that the hash isn't matched to any malware, I determined the owner of the file, HeavyLoad.exe by running `dir /q c:\Users\HeavyLoad.exe.`

<img width="787" height="152" alt="image" src="https://github.com/user-attachments/assets/5a26c49c-e225-4762-b139-bdadc1e04a3d" />

## Archiving the Suspicious File

I then ran `tar -c -a -f "C:\HeavyLoad.zip" "C:\Users\HeavyLoad.exe" "C:\Users\HeavyLoad-Hash.txt"` to archive the original file and its hash for digital forensics. 

## Quarantining the Archive (Using Netcat & Powershell) 

I logged into Kali Linux as an administrator and used Netcat to open a listening port to receive a connection from the virtual PC. Once the connection was established, the data transferred from the virtual PC was saved into a file named HeavyLoad.zip on Kali. 

### Observations & Troubleshooting Notes
- Attempted to transfer the suspicious archive (`HeavyLoad.zip`) from the Windows VM to the Kali “quarantine” system using Netcat.
- Listener command used: `nc -l -p 1234 > HeavyLoad.zip`
- The session did not establish properly; terminal returned escaped control codes (^[^[[A^[[A...) instead of file transfer data.
- Tested connection twice with same result. Suspect sandbox environment blocking or timing mismatch between listener and sender.
- MetaDefender scan link also failed (likely due to invalid or expired commercial license).
- Switched to alternate method in playbook (WinSCP) to complete the quarantine step successfully.

- <img width="1217" height="603" alt="image" src="https://github.com/user-attachments/assets/29187557-6ca7-498f-9af7-4737702b8627" />

<img width="935" height="624" alt="image" src="https://github.com/user-attachments/assets/4d2f62fb-61ee-4e91-8f58-c0dddd1c3868" />

If you look closely, I did not transfer the file properly. It's showing as O bytes. I missed this the first time around. 

<img width="932" height="622" alt="image" src="https://github.com/user-attachments/assets/fa423b6d-0a77-4a52-b6e3-85e15db71a06" />

As a result, Kali could not unzip the file. It's important to be mindful while implementing playbooks and SOPs. Small details like this can stall efficiency. 

<img width="804" height="370" alt="image" src="https://github.com/user-attachments/assets/5e2ff279-6259-459d-894f-ab07faef71b9" />

I checked other directories to make sure I had not mistakenly uploaded the suspicious file anywhere else. 

<img width="650" height="521" alt="image" src="https://github.com/user-attachments/assets/413deccb-4f79-4873-9577-5412a3a03c09" /> <br>


I later logged back into the virtual PC and uploaded the suspicious file to the correct directory. <br>

<img width="744" height="500" alt="image" src="https://github.com/user-attachments/assets/10353b95-54db-49b1-92b9-7cd10cf1915c" /> <br>

<img width="649" height="174" alt="image" src="https://github.com/user-attachments/assets/460ca966-d4be-4cf1-b920-a9e852613624" /> <br>

## Eradicating the Suspicious File

In the CLI, I deleted the file from both `C:` drive and user directory <br>
Using the native del command will delete the files, but it is not secure destruction of the files' contents. An undelete operation (using a third-party tool) can recover access to deleted files whose storage areas are not yet overwritten. Also, the CLI del command deletes files directly rather than sending them to the Recycle Bin. <br>

## Incident Report

Though the simulation does not require a report, it did walk me through the core components to include in a report. It basically walked me through the playbook addressing a high-CPU IR. It also further noted that this kind of report is known as an **AAR (After Action Report)**. <br>

It can also be referred to as a **Lessons Learned** or **Post-Mortem report**. The goal or purpose of this report is to document the activities performed, note any discrepancies or problems encountered, and glean information about where the process, playbook, toolset, or environment may need to be changed or improved. <br>

Once the report is crafted, it should be submitted to your CISO (Chief Information Security Officer) for review.

This lab further solidifies the Incident Response Life Cycle: 

1.Prepare 
2. Detect 
3. Analyze 
4. Contain (Quarantine) 
5. Eradicate 
6. Recovery 
7. Lessons Learned 


















