# [[PoisonedCredentials](https://cyberdefenders.org/blueteam-ctf-challenges/poisonedcredentials/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Network Forensics |
| Difficulty | Easy |
| Completed | 2026/04/29 |

## Scenario
The organization's security team has detected a surge in suspicious network activity.<br><br>
There are concerns that LLMNR (Link-Local Multicast Name Resolution) and NBT-NS (NetBIOS Name Service) poisoning attacks may be occurring within the network.<br><br>
These attacks are known for exploiting these protocols to intercept network traffic and potentially compromise user credentials.<br><br>
The task is to investigate the network logs and examine captured network traffic.

## Tools Used
- WireShark

## Investigation
### Step1: Identify the first victim
#### Q1: In the context of the incident described in the scenario, the attacker initiated their actions by taking advantage of benign network traffic from legitimate machines. Can you identify the specific mistyped query made by the machine with the IP address 192.168.232.162?
Opened Wireshark and searched the IP address and the query with the protocols (LLMNR and NBT-NS).<br><br>
Packets contained the mistyped query "FILESHAARE".<br><br>
<img width="1035" height="665" alt="image" src="https://github.com/user-attachments/assets/6c971400-8d58-4bda-85bb-24376d7d2272" />
<br><br>
A. FILESHAARE
<br><br>
#### Q2: We are investigating a network security incident. To conduct a thorough investigation, We need to determine the IP address of the rogue machine. What is the IP address of the machine acting as the rogue entity?
Widened the query to capture both requests and responses with the above IP address.<br><br>
```
ip.addr == 192.168.232.162 and (nbns or llmnr)
```
<br><br>
Then, one IP address responded to the IP address.<br><br>
<img width="1387" height="367" alt="image" src="https://github.com/user-attachments/assets/cec71ed2-a88d-4355-b703-c99929976822" />
<br><br>
The rogue machine responds to typo query to steal credentials.<br><br>

A. 192[.]168[.]232[.]215
<br><br>
### Step2: Pivot to other machines
#### Q3: As part of our investigation, identifying all affected machines is essential. What is the IP address of the second machine that received poisoned responses from the rogue machine?
Changed the query to find packets from the rogue machine.<br><br>
```
ip.src == 192.168.232.215 and (nbns or llmnr)
```
<br><br>
<img width="1051" height="373" alt="image" src="https://github.com/user-attachments/assets/9b5dea8c-44fc-47b8-a92a-582d0460674d" />
<br><br>
A. 192[.]168[.]232[.]176
<br><br>
#### Q4: We suspect that user accounts may have been compromised. To assess this, we must determine the username associated with the compromised account. What is the username of the account that the attacker compromised?
Even widened the search. Investigated packets related to the rogue IP address, and found SMB sessions.<br><br>
User name is janesmith.<br><br>
<img width="1448" height="253" alt="image" src="https://github.com/user-attachments/assets/3a2ce57e-65ec-4508-b711-55322c18a538" />
<br><br>
A. janesmith
<br><br>
#### Q5: As part of our investigation, we aim to understand the extent of the attacker's activities. What is the hostname of the machine that the attacker accessed via SMB?
Investigated the response from the second compromised IP to the rogue IP with SMB protocol.<br><br>
NTLMSSP CHALLENGE contains useful information such as target info, including the hostname of the machine.<br><br>
<img width="1656" height="731" alt="image" src="https://github.com/user-attachments/assets/7d8ddad5-3d07-447a-94ad-68f54ecffc10" />
<br><br>
A. AccountingPC
<br><br>
## Key Findings / IOCs
| Type | Value |
|---|---|
| Typo Query | FILESHAARE |
| Rogue Machine (compromised) | 192[.]168[.]232[.]215 |
| Second Victim | 192[.]168[.]232[.]176 |
| Compromised User | janesmith |
| Host | AccountingPC |


## Lessons Learned
- I'd learned that LLMNR and NBT-NS attacks are often observed once attackers gain the internal access and try to establish persistence, via SEC504. This lab is a good example how the attacker compromised SMB user access via these protocols.
- Finding the hostname of the machine was tough. NTLMSSP Challenge includes a lot of information since the response needs them, which helped find the answer. 
