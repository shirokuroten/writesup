# [[WebStrike](https://cyberdefenders.org/blueteam-ctf-challenges/webstrike/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Network Forensics |
| Difficulty | Easy |
| Completed | 2026/04/29 |

## Scenario
A suspicious file was identified on a company web server, raising alarms within the intranet. <br><br>
The Development team flagged the anomaly, suspecting potential malicious activity. To address the issue, the network team captured critical network traffic and prepared a PCAP file for review.<br><br>
The task is to analyze the provided PCAP file to uncover how the file appeared and determine the extent of any unauthorized activity.

## Tools Used
- Wireshark

## Investigation

### Step1: Understand the attacker's infrastructure

#### Q1: Identifying the geographical origin of the attack facilitates the implementation of geo-blocking measures and the analysis of threat intelligence. From which city did the attack originate?
Opened the PCAP file with Wireshark, and found that there were only two IP addresses in the file.<br><br>
<img width="803" height="167" alt="image" src="https://github.com/user-attachments/assets/301446b7-e188-452f-80a3-7ea564ce4744" />
<br><br>
`http.request.method == "GET"` showed that one IP address sent requests to the others.<br><br>
<img width="1149" height="381" alt="image" src="https://github.com/user-attachments/assets/63a95de1-4e19-4218-8a8d-d738ac23c84e" />
<br><br>
Looked up this IP address and found the city name.<br><br>
<img width="1103" height="690" alt="image" src="https://github.com/user-attachments/assets/8302c04c-95d2-4954-abe5-d7faf338cf9c" />
<br><br>

A. Tianjin
<br><br>
#### Q2: Knowing the attacker's User-Agent assists in creating robust filtering rules. What's the attacker's Full User-Agent?
Analyzed HTTP packet and found the User-Agent the attacker used.<br><br>
<img width="1140" height="476" alt="image" src="https://github.com/user-attachments/assets/9e2ccdc9-2d3e-4873-b429-e141a4ac22f2" />
<br><br>
A. Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
<br><br>
### Step2: Detect exploitation

#### Q3: We need to determine if any vulnerabilities were exploited. What is the name of the malicious web shell that was successfully uploaded?
Investigated HTTP packets via `http.request.method == "POST"`. Found a php file uploaded.<br><br>
<img width="1192" height="110" alt="image" src="https://github.com/user-attachments/assets/6232037a-9c4e-408c-9237-8571e9e9424f" />
<br><br>
A. image.jpg.php
<br><br>
#### Q4: Identifying the directory where uploaded files are stored is crucial for locating the vulnerable page and removing any malicious files. Which directory is used by the website to store the uploaded files?
Investigated the same packet and found the directory.<br><br>

A. /reviews/uploads/
<br><br>
#### Q5: Which port, opened on the attacker's machine, was targeted by the malicious web shell for establishing unauthorized outbound communication?
Opened Follow HTTP Stream to see the data of the POST packet.<br><br>
Found a php script with the port number.<br><br>
<img width="1285" height="849" alt="image" src="https://github.com/user-attachments/assets/d3f939b8-cf53-41f0-af9e-363b0617675c" />
<br><br>
A. 8080
<br><br>
#### Q6: Recognizing the significance of compromised data helps prioritize incident response actions. Which file was the attacker attempting to exfiltrate?
Investigated the packet from the internal IP to the attacker IP.<br><br>
The attacker tried to exfiltrate /etc/passwd.<br><br>
<img width="927" height="203" alt="image" src="https://github.com/user-attachments/assets/20630d29-992a-46d6-a740-0fcd6aca93ad" />
<br><br>
A. /etc/passwd
<br><br>
## Key Findings / IOCs
| Type | Value |
|---|---|
| City | Tianjin |
| IP | 117[.]11[.]88[.]124 |
| User Agent | Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0 |
| Web Shell | image.jpg.php |
| Directory | /reviews/uploads/ |
| Targeted Port | 8080 |
| Exfiltrated file | /etc/passwd |

## Lessons Learned
- Capturing User Agent is so important since IP address is easily changed, but user agent is a bit difficult to change if the attacker uses the same automation system. It's better to combine with TLS fingerprint to make it more robust.
- "Follow Stream" is very useful to investigate the payload of the specific packet when using Wireshark.
