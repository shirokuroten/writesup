# [[Volatility Traces](https://cyberdefenders.org/blueteam-ctf-challenges/volatility-traces/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Endpoint Forensics |
| Difficulty | Easy |
| Completed | 2026/04/30 |

## Scenario
On May 2, 2024, a multinational corporation identified suspicious PowerShell processes on critical systems, indicating a potential malware infiltration.<br>
This activity poses a threat to sensitive data and operational integrity.

I have been provided with a memory dump (memory.dmp) from the affected system.<br>
The task is to analyze the dump to trace the malware's actions, uncover its evasion techniques, and understand its persistence mechanisms.

## Tools Used
- Volatility3

## Investigation
### Step1: Understand parent and child processes
#### Q1: Identifying the parent process reveals the source and potential additional malicious activity. What is the name of the suspicious process that spawned two malicious PowerShell processes?
I was struggling to find the parent process with `pstree` plugin since it didn't show up the parent for powershell.<br><br>
```
./vol.py -f ../../Artifacts/memory.dmp windows.pstree
```
<img width="2411" height="173" alt="image" src="https://github.com/user-attachments/assets/f6628c79-2f44-4772-8a99-fe2e108f31bb" />
<br><br>
I changed my strategy. Ran with `psscan` instead.<br><br>
Parent process ID is 4596.<br><br>
<img width="854" height="123" alt="image" src="https://github.com/user-attachments/assets/9cc7d526-e6ec-4b25-bc1a-0e77dc225fa1" />
<br><br>
The process is InvoiceCheckList.exe.<br><br>
<img width="507" height="86" alt="image" src="https://github.com/user-attachments/assets/2df871f9-2b41-4cc6-a221-533c89fe8b71" />
<br><br>
A.InvoiceCheckList.exe
<br><br>

#### Q2: By determining which executable is utilized by the malware to ensure its persistence, we can strategize for the eradication phase. Which executable is responsible for the malware's persistence?
Still investigated the process related to the above parent process.<br><br>
```
./vol.py -f ../../Artifacts/memory.dmp windows.psscan | grep 4596
```
The parent process spawned several processes. schtasks.exe is often used for persistence.<br><br>
<img width="849" height="278" alt="image" src="https://github.com/user-attachments/assets/e2586a3d-420c-4927-970e-291de8b909db" />
<br><br>
A. schtasks.exe
<br><br>

#### Q3: Understanding child processes reveals potential malicious behavior in incidents. Aside from the PowerShell processes, what other active suspicious process, originating from the same parent process, is identified?
Investigated the above screenshot. Some RegSvcs.exe were spawned.<br><br>

A. RegSvcs.exe
<br><br>

### Step2: Identify Defense Evasion Technique
#### Q4: Analyzing malicious process parameters uncovers intentions like defense evasion for hidden, stealthy malware. What PowerShell cmdlet used by the malware for defense evasion?
Analyzed command line by `./vol.py -f ../../Artifacts/memory.dmp windows.cmdline`.<br><br>
Two powershell command lines use the same cmdlet, Add-MpPreference.<br><br>
<img width="2740" height="99" alt="image" src="https://github.com/user-attachments/assets/7226576b-4b11-4398-8d1b-52003ff2f2ed" />
<br><br>
A. Add-MpPreference
<br><br>
Add-MpPreference allows to add files, processes to an exclude list from scanning.<br>
<br><br>
#### Q5: Recognizing detection-evasive executables is crucial for monitoring their harmful and malicious system activities. Which two applications were excluded by the malware from the previously altered application's settings?
Based on the above analysis, two applications were excluded.<br>

A. InvoiceCheckList.exe,HcdmIYYf.exe

#### Q6: What is the specific MITRE sub-technique ID associated with PowerShell commands that aim to disable or modify antivirus settings to evade detection during incident analysis?
Recently (as of 2026/04/30), MITRE ATT&CK version has changed.<br><br>
They announced [the structural change](https://medium.com/mitre-attack/attack-v19-ff329cb65d66) for defense evasion.<br><br>
<img width="1039" height="351" alt="image" src="https://github.com/user-attachments/assets/be9d274f-9f5f-42a8-86a7-ce2462a8c9ba" />
<br><br>
Based on [MITRE ATT&CK v18](https://attack.mitre.org/versions/v18/techniques/T1562/001/), T1562.001 was "Impair Defenses: Disable or Modify Tools", which was the answer for the question.<br>
<br><br>
<img width="1332" height="420" alt="image" src="https://github.com/user-attachments/assets/ee959e59-8b9b-4704-8f02-afff5ac6b474" />
<br><br>
A. T1562.001
<br><br>

#### Q7: Determining the user account offers valuable information about its privileges, whether it is domain-based or local, and its potential involvement in malicious activities. Which user account is linked to the malicious processes?
To identify which user account owns a specific process, `getsids` is an effective option.<br><br>
It lists the Security Identifiers (SIDs) associated with process tokens.<br><br>

Found the account "Lee".<br><br>
```
./vol.py -f ../../Artifacts/memory.dmp windows.getsids | grep powershell
```
<br><br>
<img width="2314" height="941" alt="image" src="https://github.com/user-attachments/assets/b818a5ec-e3b3-45b9-9349-a17f8db8d425" />
<br><br>
A. Lee
<br><br>

## Key Findings / IOCs
| Type | Value |
|---|---|
| Parent Process | InvoiceCheckList.exe (4596) |
| Child Processes | Powershell.exe, RegSvcs.exe, schtasks.exe |
| Persistence | schtasks.exe |
| Defense Evasion Technique | Add-MpPreference |
| MITRE ATT&CK (v18) | Impair Defenses: Disable or Modify Tools (T1562.001) |
| User Account | Lee |

## Lessons Learned
- `pstree` is not all-around. In this lab, this didn't show the parent process for powershell. Combined with other plugins, such as `psscan`, is very crucial; not relying on single source.
- The parent process spawned persistence (schtasks.exe), defense evasion (powershell w/ Add-MpPreference), and potential execution (RegSvcs.exe as lotl). Attacker lifecyle is not alway linear; sometimes simultaneously.
