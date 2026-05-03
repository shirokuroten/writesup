# [[Psexec Hunt](https://cyberdefenders.org/blueteam-ctf-challenges/psexec-hunt/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Network Forensics |
| Difficulty | Easy |
| Completed | |

## Scenario
An alert from the Intrusion Detection System (IDS) flagged suspicious lateral movement activity involving PsExec.<br><br>
This indicates potential unauthorized access and movement across the network. <br><br>
As a SOC Analyst, the task is to investigate the provided PCAP file to trace the attacker’s activities.<br><br>
Identify their entry point, the machines targeted, the extent of the breach, and any critical indicators that reveal their tactics and objectives within the compromised environment.
<br><br>

## Tools Used
- Wireshark

## Investigation
### Step1: Understand the first pivoting
#### Q1: To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?
Investigated the log, and found SMB2 request from 10[.]0[.]0[.]130.<br><br>
<img width="2426" height="144" alt="image" src="https://github.com/user-attachments/assets/aec380cf-f828-4605-887b-7a676714e689" />
<br><br>
A. 10[.]0[.]0[.]130
<br><br>
#### Q2: To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?
Investigated the response from pivoted machine. found SALES-PC under Target Info.<br><br>
<img width="1902" height="911" alt="image" src="https://github.com/user-attachments/assets/73aa325f-c1bd-4582-b7ef-6a8f34641789" />
<br><br>
A. SALES-PC
<br><br>

#### Q3: Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?
Continued to analyze the packets. Found the username.<br><br>
<img width="981" height="129" alt="image" src="https://github.com/user-attachments/assets/0d2084ec-7ade-4726-8d37-394069e94ed9" />
<br><br>
A. ssales
<br><br>
#### Q4: After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?
Narrowed down to the packets with the pivoted machine, `ip.src == 10.0.0.133`.<br><br>
Found a write response.<br><br>
<img width="2032" height="114" alt="image" src="https://github.com/user-attachments/assets/342d2dbc-7880-464a-be26-ad010bafb3b4" />
<br><br>
A. PSEXESVC.exe
<br><br>
#### Q5: We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine?
Continued to analyze SMB packets. Found the tree (ADMIN$).<br><br>
<img width="2034" height="851" alt="image" src="https://github.com/user-attachments/assets/ee703261-4955-4727-b7ba-bf0c6bd7c482" />
<br><br>

A. ADMIN$
<br><br>
#### Q6: We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?
Investigated packets with stdin, stdout or stderr. Found packets with the tree (IPC$).<br><br>
<img width="2379" height="988" alt="image" src="https://github.com/user-attachments/assets/be2d435a-5345-4c0e-b786-c7d7baaa6205" />
<br><br>
A. IPC$
<br><br>
### Step2: Understand the second pivoting
#### Q7: Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?
Narrowed down the packets with `ip.src == 10.0.0.130 and ip.dst != 10.0.0.133`.<br><br>
Found the NTLMv2 Response in SMB2 packets.<br><br>
<img width="2337" height="918" alt="image" src="https://github.com/user-attachments/assets/534cedb0-902b-486c-b11a-b68136903faa" />
<br><br>
A. MARKETING-PC
<br><br>

## Key Findings / IOCs
| Type | Value |
|---|---|
| First Compromised Machine | 10[.]0[.]0[.]130 |
| Second Compromised Machine | 10[.]0[.]0[.]133 (SALES-PC) |
| Third Compromised Machine | 10[.]0[.]0[.]131 ( MARKETING-PC) |
| Executable | PSEXESVC.exe |

## Lessons Learned
- Quite similar with [PoisonedCredentials](https://github.com/shirokuroten/writesup/blob/main/cyberdefenders/network_forensics/poisonedcredentials.md), learned pivoting with SMB share.
- SMB is still used across many industries, but so insecure. Understanding risks and making actions if necessary are very important.
