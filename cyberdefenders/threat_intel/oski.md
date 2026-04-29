# [[Oski](https://cyberdefenders.org/blueteam-ctf-challenges/oski/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Threat Intel |
| Difficulty | Easy |
| Completed | 2026/04/29 |

## Scenario
The accountant at the company received an email titled "Urgent New Order" from a client late in the afternoon. <br>
When he attempted to access the attached invoice, he discovered it contained false order information. Subsequently, the SIEM solution generated an alert regarding downloading a potentially malicious file.<br>
Upon initial investigation, it was found that the PPT file might be responsible for this download. <br>
Need to conduct a detailed examination of this file.


## Tools Used
- VirusTotal
- ANY.RUN

## Investigation
### Step1: Examine malware hash
Opened the downloaded file and found the MD5 hash (12c1842c3ccafe7408c23ebf292ee3d9).

#### Q1: Determining the creation time of the malware can provide insights into its origin. What was the time of malware creation?
Put the hash into VirusTotal, and found the creation time at Details tab.<br>
<img width="1121" height="842" alt="image" src="https://github.com/user-attachments/assets/c8d929bd-1500-4b68-9e50-1c996b37c2ce" />

A. 2022-09-28 17:40

#### Q2: Identifying the command and control (C2) server that the malware communicates with can help trace back to the attacker. Which C2 server does the malware in the PPT file communicate with?
Moved to Relation tab and found the C2 server.<br>
<img width="1217" height="403" alt="image" src="https://github.com/user-attachments/assets/2e5738ff-3c66-4c0b-8888-96bdfc9acdea" />

A. hxxp[://]171[.]22[.]28[.]221/5c06c05b7b34e8e6[.]php 

#### Q3: Identifying the initial actions of the malware post-infection can provide insights into its primary objectives. What is the first library that the malware requests post-infection?
Moved to Behavior tab and investigated dropped files.<br>
<img width="919" height="418" alt="image" src="https://github.com/user-attachments/assets/3ce13890-a5a2-40c4-9fe8-06c0d30f56fe" />

A. sqlite3.dll

### Step2: Examine malware details by Any.run report
#### Q4: By examining the provided Any.run report, what RC4 key is used by the malware to decrypt its base64-encoded string?
Examined the Any.run report, and found the RC4 key.<br>
<img width="733" height="400" alt="image" src="https://github.com/user-attachments/assets/35792695-21b2-4369-947d-3a25eb2223e4" />

A. 5329514621441247975720749009

#### Q5: By examining the MITRE ATT&CK techniques displayed in the Any.run sandbox report, identify the main MITRE technique (not sub-techniques) the malware uses to steal the user’s password.
Opened the sandbox report, and moved to ATT&CK.<br>
<img width="795" height="279" alt="image" src="https://github.com/user-attachments/assets/bfc69900-c416-409f-b4e4-053e90403d57" />
The matrix had several techniques related to credential access.<br>
<img width="1822" height="717" alt="image" src="https://github.com/user-attachments/assets/29bcb83c-8110-41f9-a506-20888a575ed1" />

Steal Web Session Cookie and Unsecured Credentials may not be the first choice because we need to consider password stealing.<br>
"Credentials from Password Stores" should be the first choice in this case.<br>

A. T1555

#### Q6: By examining the child processes displayed in the Any.run sandbox report, which directory does the malware target for the deletion of all DLL files?
Came back to the top page, and investigated the child process (cmd.exe).<br>
The malware attempted to delete all DLL files at C:\ProgramData before exit.<br>
<img width="769" height="257" alt="image" src="https://github.com/user-attachments/assets/45c41e86-2ead-4fb4-be88-21a4cfba848d" />

A. C:\ProgramData

#### Q7: Understanding the malware's behavior post-data exfiltration can give insights into its evasion techniques. By analyzing the child processes, after successfully exfiltrating the user's data, how many seconds does it take for the malware to self-delete?
Still investigated the child process. It set the timeout (delay) before self-deletion.<br>
<img width="378" height="252" alt="image" src="https://github.com/user-attachments/assets/270235f7-e04a-4876-b6f5-3894d7b365fc" />

A. 5

## Key Findings / IOCs
| Type | Value |
|---|---|
| MD5 Hash | 12c1842c3ccafe7408c23ebf292ee3d9 |
| Creation Date | 2022-09-28 17:40 |
| C2 Server | hxxp[://]171[.]22[.]28[.]221/5c06c05b7b34e8e6[.]php |
| First Downloaded Library | sqlite3.dll |
| RC4 key | 5329514621441247975720749009 |
| Technique | Credentials from Password Stores (T1555) |
| DLL files deletion | C:\ProgramData |
| Self-Deletion Delay (seconds) | 5 |

## Lessons Learned
- VirusTotal offers rich information such as dropped files, which helps track their whole procedures (in this case, after establishing C2, downloaded sqlite3.dll).
- Downloading Sqlite3.dll indicates that they attempted to exfiltrate user and password from the database. We need to investigate the safety of the database.

