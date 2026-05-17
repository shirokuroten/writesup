# [[3CX Supply Chain](https://cyberdefenders.org/blueteam-ctf-challenges/3cx-supply-chain/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Threat Intel |
| Difficulty | Easy |
| Completed | 2026/5/15 |

## Scenario
A large multinational corporation heavily relies on the 3CX software for phone communication, making it a critical component of their business operations.<br><br>
After a recent update to the 3CX Desktop App, antivirus alerts flag sporadic instances of the software being wiped from some workstations while others remain unaffected.<br><br>
Dismissing this as a false positive, the IT team overlooks the alerts, only to notice degraded performance and strange network traffic to unknown servers.<br><br>
Employees report issues with the 3CX app, and the IT security team identifies unusual communication patterns linked to recent software updates.<br><br>
As the threat intelligence analyst, it's your responsibility to examine this possible supply chain attack.<br><br>
Your objectives are to uncover how the attackers compromised the 3CX app, identify the potential threat actor involved, and assess the overall extent of the incident. <br><br>

## Tools Used


## Investigation
### Step1: Understand Malware
#### Q1: Understanding the scope of the attack and identifying which versions exhibit malicious behavior is crucial for making informed decisions if these compromised versions are present in the organization. How many versions of 3CX running on Windows have been flagged as malware?
Google Search with `3CXDesktopApp malware` and found the report by [Fortinet](https://www.fortinet.com/blog/threat-research/3cx-desktop-app-compromised).<br><br>
Found 2 versions (versions 18.12.407 and 18.12.416).<br><br>
<img width="1115" height="187" alt="image" src="https://github.com/user-attachments/assets/82edac5b-1164-4007-93f3-799146b6a048" />
<br><br>
A. 2
<br><br>

#### Q2: Determining the age of the malware can help assess the extent of the compromise and track the evolution of malware families and variants. What's the UTC creation time of the .msi malware?
Uploaded the extracted file to VirusTotal, and found the creation time.<br><br>
<img width="581" height="230" alt="image" src="https://github.com/user-attachments/assets/6bb41fe7-9c05-47e4-9d42-7928ad3db7e6" />
<br><br>
A. 2023-03-13 06:33
<br><br>

#### Q3: Executable files (.exe) are frequently used as primary or secondary malware payloads, while dynamic link libraries (.dll) often load malicious code or enhance malware functionality. Analyzing files deposited by the Microsoft Software Installer (.msi) is crucial for identifying malicious files and investigating their full potential. Which malicious DLLs were dropped by the .msi file?
Investigated the fortinet report, and found two DLLs.<br><br>
<img width="1134" height="139" alt="image" src="https://github.com/user-attachments/assets/8456bc53-595e-45f9-83b0-08e6364553c6" />
<br><br>
A. ffmpeg.dll,d3dcompiler_47.dll
<br><br>

### Step2: Understand TTPs
#### Q4: Recognizing the persistence techniques used in this incident is essential for current mitigation strategies and future defense improvements. What is the MITRE Technique ID employed by the .msi files to load the malicious DLL?
Adversaries leveraged their own malicious DLLs. It's `Hijack Execution Flow: DLL`.
<br><br>
A. T1574 (T1754.001)
<br><br>

#### Q5: Recognizing the malware type (threat category) is essential to your investigation, as it can offer valuable insight into the possible malicious actions you'll be examining. What is the threat category of the two malicious DLLs?
These DLLs loads additional payloads.<br><br>
In VirusTotal, threat categories are trojan, pua or dropper.<br><br>
<img width="268" height="47" alt="image" src="https://github.com/user-attachments/assets/30d4546f-1326-4067-8fd7-a472b3582d88" />
<br><br>
A. trojan
<br><br>

#### Q6: As a threat intelligence analyst conducting dynamic analysis, it's vital to understand how malware can evade detection in virtualized environments or analysis systems. This knowledge will help you effectively mitigate or address these evasive tactics. What is the MITRE ID for the virtualization/sandbox evasion techniques used by the two malicious DLLs?
Searched `Virtualization Evasion` at MITRE ATT&CK and found Virtualization/Sandbox Evasion (T1497).<br><br>
<img width="664" height="238" alt="image" src="https://github.com/user-attachments/assets/79682377-4c04-45c6-beb8-d5071270a7ef" />
<br><br>
A. T1497
<br><br>

#### Q7: When conducting malware analysis and reverse engineering, understanding anti-analysis techniques is vital to avoid wasting time. Which hypervisor is targeted by the anti-analysis techniques in the ffmpeg.dll file?
Investigated the hash of ffmpeg.dll. Under Capabilities in Behavior tab, found the anti-analysis technique for VMWare.<br><br>
<img width="428" height="302" alt="image" src="https://github.com/user-attachments/assets/43ddd8ad-355e-4df3-8f38-a74a525eb626" />
<br><br>
A. VMWare
<br><br>

#### Q8: Identifying the cryptographic method used in malware is crucial for understanding the techniques employed to bypass defense mechanisms and execute its functions fully. What encryption algorithm is used by the ffmpeg.dll file?
Investigated [another report](https://www.splunk.com/en_us/blog/security/splunk-insights-investigating-the-3cxdesktopapp-supply-chain-compromise.html) by Splunk.<br><br>
Found the encryption algorithm.<br><br>
<img width="769" height="161" alt="image" src="https://github.com/user-attachments/assets/6ca1954f-07ad-4a5e-b8c6-db3c169e463d" />
<br><br>
A. RC4
<br><br>
#### Q9: As an analyst, you've recognized some TTPs involved in the incident, but identifying the APT group responsible will help you search for their usual TTPs and uncover other potential malicious activities. Which group is responsible for this attack?
Moved to MITRE ATT&CK and found a group under the campaign of 3CX Supply Chain Attack.<br><br>
<img width="991" height="527" alt="image" src="https://github.com/user-attachments/assets/678a4a97-8b82-4eb9-b4b5-76d92676d8fc" />
<br><br>
AppleJeus is a North Korean state-sponsored threat group, associated with the broader Lazarus Group.<br><br>
The answer format should start with "L", so the answer is Lazarus.<br><br>
A. Lazarus
<br><br>

## Key Findings / IOCs
| Type | Value |
|---|---|
| Compromised Windows Versions | 18.12.407 and 18.12.416 |
| DLL | ffmpeg.dll, d3dcompiler_47.dll |
| Malware Type | Trojan |
| Execution | Hijack Execution Flow: DLL (T1574.001) |
| Evasion | Virtualization/Sandbox Evasion (T1497) |
| APT | Lazarus |

## Lessons Learned
- This lab contains from malware behavior to TTPs and APT, from which I learned the whole lifecycle of CTI.
- Each vendor has different name conventions. For instance, Mandiant [assesses with moderate confidence](https://cloud.google.com/blog/topics/threat-intelligence/3cx-software-supply-chain-compromise) that UNC4736 is related to financially motivated North Korean “AppleJeus” activity as reported by CISA. Understanding what the name indicates is so important to link dots.
