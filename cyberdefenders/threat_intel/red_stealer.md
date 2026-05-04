# [[Red Stealer](https://cyberdefenders.org/blueteam-ctf-challenges/red-stealer/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Threat Intel |
| Difficulty | Easy |
| Completed | 2026/05/04 |

## Scenario
You are part of the Threat Intelligence team in the SOC (Security Operations Center).<br><br>
An executable file has been discovered on a colleague's computer, and it's suspected to be linked to a Command and Control (C2) server, indicating a potential malware infection.<br><br>
The task is to investigate this executable by analyzing its hash.<br><br>
The goal is to gather and analyze data beneficial to other SOC members, including the Incident Response team, to respond to this suspicious behavior efficiently.

## Tools Used
- VirusTotal
- MalwareBazaar
- ThreatFox

## Investigation
### Step1: Understand the Malware
#### Q1: Categorizing malware enables a quicker and clearer understanding of its unique behaviors and attack vectors. What category has Microsoft identified for that malware in VirusTotal?
Searched the malware hash at VirusTotal. Found the Microsoft analysis.<br><br>
<img width="1597" height="290" alt="image" src="https://github.com/user-attachments/assets/173542ec-1340-487e-be86-6e213b18ac61" />
<br><br>
A. Trojan
<br><br>

#### Q2: Clearly identifying the name of the malware file improves communication among the SOC team. What is the file name associated with this malware?
Moved to Details tab, and looked at "Names" area. Found the file name.<br><br>
<img width="829" height="281" alt="image" src="https://github.com/user-attachments/assets/e0944687-aecf-4751-a84c-521f67b7ba01" />
<br><br>
A. Wextract
<br><br>
#### Q3: Knowing the exact timestamp of when the malware was first observed can help prioritize response actions. Newly detected malware may require urgent containment and eradication compared to older, well-documented threats. What is the UTC timestamp of the malware's first submission to VirusTotal?
Still investigated Details tab. Found the first submission date and time.<br><br>
<img width="827" height="326" alt="image" src="https://github.com/user-attachments/assets/fb363071-94b8-46db-8614-e16e784ad58a" />
<br><br>
A. 2023-10-06 04:41:50
<br><br>
### Step2: Understand TTPs
#### Q4: Understanding the techniques used by malware helps in strategic security planning. What is the MITRE ATT&CK technique ID for the malware's data collection from the system before exfiltration?
Moved to Behavior tab, MITRE ATT&CK matrix. Found the technique (Data from Local System) under Data Collection.<br><br>
<img width="1160" height="684" alt="image" src="https://github.com/user-attachments/assets/c06723b1-4567-4129-99e1-33757cb62774" />
<br><br>
A. T1005
<br><br>
#### Q5: Following execution, which social media-related domain names did the malware resolve via DNS queries?
Investigated DNS resolutions history. Found "facebook.com".<br><br>
<img width="836" height="462" alt="image" src="https://github.com/user-attachments/assets/6ec1812b-9fa1-4b47-a0a5-2c01faef24cf" />
<br><br>
A. facebook.com
<br><br>
#### Q6: Once the malicious IP addresses are identified, network security devices such as firewalls can be configured to block traffic to and from these addresses. Can you provide the IP address and destination port the malware communicates with?
Investigated IP traffic. Found several IPs and discovered that one of them seemed malicious.<br><br>
<img width="812" height="545" alt="image" src="https://github.com/user-attachments/assets/f6974350-fb97-4878-9a74-6a98667a73f0" />
<br><br>
<img width="1021" height="394" alt="image" src="https://github.com/user-attachments/assets/23f0b2d7-18ea-488c-8439-acc7f6be13ef" />
<br><br>
A. 77.91.124.55:19071
<br><br>
#### Q7: YARA rules are designed to identify specific malware patterns and behaviors. Using MalwareBazaar, what's the name of the YARA rule created by "Varp0s" that detects the identified malware?
Searched the hash at MalwareBazaar, and found several YARA rules.<br><br>
<img width="838" height="117" alt="image" src="https://github.com/user-attachments/assets/9c4361a2-c0e2-4171-8733-1b6c54fd0299" />

<br><br>
A. detect_Redline_Stealer
<br><br>
#### Q8: Understanding which malware families are targeting the organization helps in strategic security planning for the future and prioritizing resources based on the threat. Can you provide the different malware alias associated with the malicious IP address according to ThreatFox?
Searched the above malicious IP address at ThreatFox like `ioc: 77.91.124.55`. Found the alias.<br><br>
<img width="818" height="367" alt="image" src="https://github.com/user-attachments/assets/3aaf6253-f98b-4b44-b6d6-54dc0c8d2b45" />
<br><br>
A. RECORDSTEALER
<br><br>
#### Q9: By identifying the malware's imported DLLs, we can configure security tools to monitor for the loading or unusual usage of these specific DLLs. Can you provide the DLL utilized by the malware for privilege escalation?
Moved back to Detail tab in VirusTotal. There are several "Imported" DLLs.<br><br>
Under ADVAPI32.dll, found AdjustTokenPrivileges or OpenProcessToken, which are necessary for manipulating access tokens and escalating privileges.<br><br>
<img width="824" height="587" alt="image" src="https://github.com/user-attachments/assets/9d279fee-c104-4b1e-bbf0-7765af846627" />

<br><br>
A. ADVAPI32.dll
<br><br>
## Key Findings / IOCs
| Type | Value |
|---|---|
| Malware Category | Trojan |
| Hash | 248fcc901aff4e4b4c48c91e4d78a939bf681c9a1bc24addc3551b32768f907b (SHA256) |
| Malware Alias | RECORDSTEALER |
| File Name | Wextract |
| Data Collection | Data from Local System (T1005) |
| DNS Resolution Domain | facebook[.]com |
| C2 | 77[.]91[.]124[.]55:19071 |
| YARA Rule | detect_Redline_Stealer |
| DLL for Privilege Escalation | ADVAPI32.dll |

## Lessons Learned
- I was struggling to find the imported files in VirusTotal, which was found under "Details" tab, not "Behavior" tab. Based on search, this is because imported files are discoverd by static analysis.
- In static analysis, VirusTotal can find them without execution, because imported files are listed on "Import Address Table" (metadata).
