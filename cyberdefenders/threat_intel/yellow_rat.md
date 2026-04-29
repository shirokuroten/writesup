# [[Yellow RAT](https://cyberdefenders.org/blueteam-ctf-challenges/yellow-rat/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Threat Intel |
| Difficulty | Easy |
| Completed | 2026/04/29 |

## Scenario
During a regular IT security check at GlobalTech Industries, abnormal network traffic was detected from multiple workstations.<br>
Upon initial investigation, it was discovered that certain employees' search queries were being redirected to unfamiliar websites.<br>
This discovery raised concerns and prompted a more thorough investigation.<br>
The task is to investigate this incident and gather as much information as possible.

## Tools Used
- VirusTotal
- Red Canary

## Investigation
### Step1: Malware Overview
#### Q1: Understanding the adversary helps defend against attacks. What is the name of the malware family that causes abnormal network traffic?
Malware family name is different by vendors. In this lab, we need to rely on Red Canary.<br>
Unfortunately, "Detection" tab didn't show one. Moved to "Relations" tab, and clicked the graph (you need to log-in).<br>
In the graph, there were several collections (reports), and one of them was published by Red Canary.<br>
<img width="1916" height="821" alt="image" src="https://github.com/user-attachments/assets/8619f50a-b230-4df8-94aa-62241bae828d" />

[The report](https://otx.alienvault.com/pulse/5fcab7a1accb28c015a5717d) showed the malware family name.<br>
<img width="1626" height="491" alt="image" src="https://github.com/user-attachments/assets/06b6907d-789e-4e61-9d7b-e4d73d1a6505" />

A. Yellow Cockatoo RAT

#### Q2: As part of our incident response, knowing common filenames the malware uses can help scan other workstations for potential infection. What is the common filename associated with the malware discovered on our workstations?
Common filenames can be seen at Details tab.<br>
<img width="785" height="530" alt="image" src="https://github.com/user-attachments/assets/e07511d7-a6e7-42c3-b5d6-37260868769a" />

A. 111bc461-1ca8-43c6-97ed-911e0e69fdf8.dll

When searching hash in Hybrid Analysis, it actually shows the same name ([reference](https://hybrid-analysis.com/sample/30e527e45f50d2ba82865c5679a6fa998ee0a1755361ab01673950810d071c85/5f87b2920788cb226f59d611)).

#### Q3: Determining the compilation timestamp of malware can reveal insights into its development and deployment timeline. What is the compilation timestamp of the malware that infected our network?
Compilation timestamp can be found at Details tab as well.<br>
<img width="947" height="471" alt="image" src="https://github.com/user-attachments/assets/569812dc-1a56-4a05-98ff-429fa46bdc4c" />

A. 2020-09-24 18:26

#### Q4: Understanding when the broader cybersecurity community first identified the malware could help determine how long the malware might have been in the environment before detection. When was the malware first submitted to VirusTotal?
Investigated Detail tab in VirusTotal.<br>
<img width="848" height="329" alt="image" src="https://github.com/user-attachments/assets/8e399668-0368-42b2-b95c-a280057a6f72" />

A. 2020-10-15 02:47

### Step2: Malware Detail
#### Q5: To completely eradicate the threat from Industries' systems, we need to identify all components dropped by the malware. What is the name of the .dat file that the malware dropped in the AppData folder?
The report in Q1 has the link to [the detailed report](https://redcanary.com/blog/threat-intelligence/yellow-cockatoo/), which contains the .dat file name.<br>
<img width="1178" height="794" alt="image" src="https://github.com/user-attachments/assets/0bf03b1f-a3bf-441c-9186-badf71b93341" />

A. solarmarker.dat

#### Q6: It is crucial to identify the C2 servers with which the malware communicates to block its communication and prevent further data exfiltration. What is the C2 server that the malware is communicating with?
The above screenshot contains the C2 server as well.<br>

A. hxxps[://]gogohid[.]com

## Key Findings / IOCs
| Type | Value |
|---|---|
| Malware Family | Yellow Cockatoo RAT |
| Common Name | 111bc461-1ca8-43c6-97ed-911e0e69fdf8.dll |
| Compilation Date | 2020-09-24 18:26 |
| First Submission Date | 2020-10-15 02:47 |
| Dropped File | solarmarker.dat |
| Dropped Directory | %USERPROFILE%\AppData\Roaming |
| C2 Server | hxxps[://]gogohid[.]com |

## Lessons Learned
- Q1 was tricky. Malware family name is different among vendors, so from Q1, I can learn how to leverage VirusTotal Graph. In CTI field, graph based database is often used to express the relationship between each node. I know and have used it for writing reports, but this lab gave me additional experience about pivoting through data sources and visualizing relationships for reporting.
- This malware names filename based on the victim's search query. I believe it's effective since the name is familiar with the victim, which makes the file less suspicious.
