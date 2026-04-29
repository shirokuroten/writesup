# [[Lespion](https://cyberdefenders.org/blueteam-ctf-challenges/lespion/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category |　Threat Intel |
| Difficulty | Easy |
| Completed | 2026/04/29 |

## Scenario
A client network was compromised and brought offline.<br>
The task is to investigate the incident and determine the attacker's identity.<br>

Incident responders and digital forensic investigators are currently on the scene and have conducted a preliminary investigation.<br>
Their findings show that the attack originated from a single user account, probably, an insider. <br>
Investigate the incident, find the insider, and uncover the attack actions.



## Tools Used
- Google Image Search

## Investigation
### Step1: 
#### Q1: File -> Github.txt: What API key did the insider add to his GitHub repositories?
Opened [the github page](https://github.com/EMarseille99). Repositories > Project-Build---Custom-Login-Page > Login Page.js, found the API key.<br>
<img width="890" height="308" alt="image" src="https://github.com/user-attachments/assets/33d5e064-2d78-4101-b635-dc9c437ffc3c" />

A. aJFRaLHjMXvYZgLPwiJkroYLGRkNBW
<br>

#### Q2: File -> Github.txt: What plaintext password did the insider add to his GitHub repositories?
Read through Login Page.js and found Base 64 encoded password.<br>
<img width="819" height="321" alt="image" src="https://github.com/user-attachments/assets/1f0416aa-aaf7-416b-8919-d2d53a2458de" />
<br>
Decoded it at CyberChef, and got the plaintext password.<br>

A. PicassoBaguette99
<br>

#### Q3: File -> Github.txt: What cryptocurrency mining tool did the insider use?
Investigated the repositories and found the cryptocurrency mining tool.<br>
<img width="765" height="193" alt="image" src="https://github.com/user-attachments/assets/9f72fc66-0d9f-4748-9cf0-22b8792c5944" />


A. XMRig
<br>

#### Q4: On which gaming website did the insider have an account?
Google Search for the user account found at Q2.<br>
Found a steam account.<br>
<img width="1360" height="862" alt="image" src="https://github.com/user-attachments/assets/13fc446a-bd6b-4546-a00f-995bff944b9a" />

A. steam
<br>

#### Q5: What is the link to the insider Instagram profile?
The above screenshot contained the instagram account as well.<br>

A. hxxps[://]www[.]instagram[.]com/emarseille99/
<br>

#### Q6: Which country did the insider visit on her holiday?
Investigated her instagram post.<br>
She went to Marina Bay Sands.<br>
<img width="1552" height="642" alt="image" src="https://github.com/user-attachments/assets/e5c697dd-3ec9-4bd7-9b7d-ced3e66e9295" />

A. Singapore
<br>

#### Q7: Which city does the insider family live in?
Still investigated other posts. <br>
<img width="871" height="507" alt="image" src="https://github.com/user-attachments/assets/753334b4-7e2e-46e6-9c91-44182f3ea9e2" />

Based on Google lens search, it should be Dubai.

A. Dubai
<br>

#### Q8: File -> office.jpg: You have been provided with a picture of the building in which the company has an office. Which city is the company located in?
Leveraged Google Image Search to find the city. AI Overview answered "Birmingham, UK".<br>
<img width="1162" height="543" alt="image" src="https://github.com/user-attachments/assets/55ccc49b-b7cd-49c5-b55f-2835adbe5550" />

A. Birmingham
<br>

#### Q9: File -> Webcam.png: With the intel, you have provided, our ground surveillance unit is now overlooking the person of interest suspected address. They saw them leaving their apartment and followed them to the airport. Their plane took off and landed in another country. Our intelligence team spotted the target with this IP camera. Which state is this camera in?
Still leveraged Google Image Search to find the state. AI Overview also gave an answer.
<img width="1174" height="980" alt="image" src="https://github.com/user-attachments/assets/01eef7d3-fbcd-4973-95a9-a9a73810ab8d" />

A. Indiana
<br>

## Key Findings / IOCs
| Type | Value |
|---|---|
| API Key | aJFRaLHjMXvYZgLPwiJkroYLGRkNBW |
| User Name | EMarseille99 |
| Password | PicassoBaguette99 (Base64: UGljYXNzb0JhZ3VldHRlOTk=) |
| Insider's Family | Dubai |
| Insider's Current Area | Indiana |


## Lessons Learned
- This lab was not traditional cybersecurity one, but so fun. Leveraing OSINT tools such as Google Image Search widens our investigation.
- Instagram and Steam account itself might not be useful, but understanding their origins, interests and preference may be a good hint for further investigation.
