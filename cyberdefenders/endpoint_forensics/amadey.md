# [[Amadey](https://cyberdefenders.org/blueteam-ctf-challenges/amadey-apt-c-36/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Endpoint Forensics |
| Difficulty | Medium |
| Completed | 2026/04/29 |

## Scenario
An after-hours alert from the Endpoint Detection and Response (EDR) system flags suspicious activity on a Windows workstation.<br><br>
The flagged malware aligns with the Amadey Trojan Stealer.<br><br>
The job is to analyze the presented memory dump and create a detailed report for actions taken by the malware.

## Tools Used
- Volatility3

## Investigation
### Step1: Identify the Malicious Process
#### Q1: In the memory dump analysis, determining the root of the malicious activity is essential for comprehending the extent of the intrusion. What is the name of the parent process that triggered this malicious behavior?
To analyze the memory dump, we need to use volatility3.<br><br>
Volatility3 syntax is like<br><br>
```
vol.py -f dump_file plugin
```
<br><br>
Plugin allows to analyze specific areas.<br><br>
In this case, I wanted to see the parent-child relationship in Windows, so I used "windows.pstree".<br><br>
```
./vol.py -f ../../Artifacts/'Windows 7 x64-Snapshot4.vmem' windows.pstree
```
<br><br>
There are lots of parent-child, but one parent process name was suspicious (lssass, not lsass), and spawned rundll32.exe, which seems suspicious.<br><br>
<img width="799" height="49" alt="image" src="https://github.com/user-attachments/assets/4b9e50c6-fafa-44ec-b3d7-f210d00c5d83" />
<br><br>
A. lssass.exe
<br><br>
#### Q2: Once the rogue process is identified, its exact location on the device can reveal more about its nature and source. Where is this process housed on the workstation?
Google Searched the right plugin, and found "windows.dlllist", which lists the DLLs and full path of the main process.<br><br>
In my scenario, the process ID was 2748, so the syntax was like<br><br>
```
./vol.py -f ../../Artifacts/'Windows 7 x64-Snapshot4.vmem' windows.dlllist --pid 2748
```
<br><br>
It revealed several loaded modules. One of file outputs is different from others.<br><br>
<img width="1798" height="231" alt="image" src="https://github.com/user-attachments/assets/41f042be-2941-482c-8022-cb95edb22730" />
<br><br>
A. C:\Users\0XSH3R~1\AppData\Local\Temp\925e7e99c5\lssass.exe
<br><br>
### Step2: Find C2 Server
#### Q3: Persistent external communications suggest the malware's attempts to reach out C2C server. Can you identify the Command and Control (C2C) server IP that the process interacts with?
Based on help, used "windows.netscan" plugin to find network connections.<br><br>
```
./vol.py -f ../../Artifacts/'Windows 7 x64-Snapshot4.vmem' windows.netscan
```
<br><br>
Found "ForeignAddr" as a C2 server. 41[.]75[.]84[.]12 had 80 (http), which indicates active C2 communication.<br><br>
<img width="1463" height="598" alt="image" src="https://github.com/user-attachments/assets/66fc5360-eca7-4ca6-a951-a5b43f58fe0b" />
<br><br>
A. 41[.]75[.]84[.]12
<br><br>
#### Q4: Following the malware link with the C2C, the malware is likely fetching additional tools or modules. How many distinct files is it trying to bring onto the compromised workstation?
"Memmap" allows to find all the memory pages for the specific process. It includes HTTP GET request as well.<br><br>
Ran the following code to dump the memory map, then search "GET /" since I wanted to find additional tools or modules with C2. <br><br>
```
./vol.py -f ../../Artifacts/'Windows 7 x64-Snapshot4.vmem' windows.memmap --pid 2748 --dump
```
```
strings pid.2748.dmp | grep "GET /"
```
<br><br>
<img width="474" height="76" alt="image" src="https://github.com/user-attachments/assets/1aeb24ad-fcfe-4002-b987-d4ce3c659cce" />
<br><br>
A. 2
<br><br>
### Step3: Investigate Child Process
#### Q5: Identifying the storage points of these additional components is critical for containment and cleanup. What is the full path of the file downloaded and used by the malware in its malicious activity?
Investigated the command line. The child process (rundll32.exe) had the above file as a command line argument.<br><br>
```
./vol.py -f ../../Artifacts/'Windows 7 x64-Snapshot4.vmem' windows.cmdline
```
<br><br>
<img width="1540" height="123" alt="image" src="https://github.com/user-attachments/assets/17d4e6f0-b653-44f2-b3db-21dd84d8daf6" />
<br><br>
A. C:\Users\0xSh3rl0ck\AppData\Roaming\116711e5a2ab05\clip64.dll
<br><br>
#### Q6: Once retrieved, the malware aims to activate its additional components. Which child process is initiated by the malware to execute these files?
It was a very straightforward question. Based on the above, rundll32.exe was initiated by lssass.exe.<br><br>

A. rundll32.exe
<br><br>
### Step4: Identify Persistence
#### Q7: Understanding the full range of Amadey's persistence mechanisms can help in an effective mitigation. Apart from the locations already spotlighted, where else might the malware be ensuring its consistent presence?
Investigated file system to find persistence.<br><br>
Used "filescan".<br><br>
```
./vol.py -f ../../Artifacts/'Windows 7 x64-Snapshot4.vmem' windows.filescan | grep .exe
```
<br><br>
lssass.exe was under Tasks directory. It indicated the scheduled task, common persistence technique.<br><br>
<img width="861" height="174" alt="image" src="https://github.com/user-attachments/assets/b2d8c937-0587-4af6-87bb-3c38dd422a24" />
<br><br>
A. C:\Windows\System32\Tasks\lssass.exe

## Key Findings / IOCs
| Type | Value |
|---|---|
| Malware | lssass.exe |
| Location | C:\Users\0XSH3R~1\AppData\Local\Temp\925e7e99c5\lssass.exe |
| Child Process | rundll32.exe |
| C2 Server | 41[.]75[.]84[.]12 |
| Additional Tool | cred64.dll |
| Additional Tool | clip64.dll |
| Tool Location | C:\Users\0xSh3rl0ck\AppData\Roaming\116711e5a2ab05\clip64.dll |
| Scheduled Task | C:\Windows\System32\Tasks\lssass.exe |

## Lessons Learned
- This was the first time for me to work on memory forensic. I was so excited to see how rich contents memory contains (e.g. HTTP GET Request). I learned that memory dumps preserve far more investigative artifacts than I expected.
- Volatility3 is a powerful tool combined with its plugins. "Pstree" is very effective for identifying masquerading processes by revealing parent-child relationships, like Procmon.
- HTTP request history is invaluable for tracing C2 communication even without network logs.
