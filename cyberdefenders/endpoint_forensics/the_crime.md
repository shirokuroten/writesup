# WIP: [[The Crime](https://cyberdefenders.org/blueteam-ctf-challenges/the-crime/)] - CyberDefenders Writeup

## Overview
| | |
|---|---|
| Platform | CyberDefenders |
| Category | Endpoint Forensics |
| Difficulty | Easy |
| Completed | |

## Scenario
We're currently in the midst of a murder investigation, and we've obtained the victim's phone as a key piece of evidence.<br>
After conducting interviews with witnesses and those in the victim's inner circle,<br>
The objective is to meticulously analyze the information we've gathered and diligently trace the evidence to piece together the sequence of events leading up to the incident.


## Tools Used
- ALEAPP

## Investigation
### Step1: Build the environment and Extract basic info
Downloaded ALEPP and dependencies.<br><br>
```
sudo apt update
sudo apt install python3 python3-pip git -y
cd Tools
git clone https://github.com/abrignoni/ALEAPP.git
cd ALEAPP
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```
Then, opened a GUI version.<br><br>
```
python3 aleappGUI.py
```
<br><br>
Selected an extracted folder and output directory, then ran Process.<br><br>
<img width="1308" height="1126" alt="image" src="https://github.com/user-attachments/assets/c27f0496-1102-4b13-8a42-bd0df10536bf" />
<br><br>
Opened report and Analyzed it.
<img width="2525" height="1392" alt="image" src="https://github.com/user-attachments/assets/0b12fefe-c0f7-4557-ac00-898652303536" />

<br><br>

#### Q1: Based on the accounts of the witnesses and individuals close to the victim, it has become clear that the victim was interested in trading. This has led him to invest all of his money and acquire debt. Can you identify the SHA256 of the trading application the victim primarily used on his phone?
Installed Apps > Installed Apps (GMS) for user 0, Found the hash of trading app.<br><br>
<img width="2035" height="566" alt="image" src="https://github.com/user-attachments/assets/4b63827a-896c-424b-afc4-469c0d07f0a3" />

A. 4f168a772350f283a1c49e78c1548d7c2c6c05106d8b9feb825fdc3466e9df3c
<br><br>

cf, GMS: [Google Mobile Services](https://www.android.com/gms/), a collection of Google applications and APIs that help support functionality across devices
<br><br>

#### Q2: According to the testimony of the victim's best friend, he said, "While we were together, my friend got several calls he avoided. He said he owed the caller a lot of money but couldn't repay now". How much does the victim owe this person?
SMS & MMS > SMS messages, found the message.<br><br>
<img width="3110" height="858" alt="image" src="https://github.com/user-attachments/assets/24d0b9fc-9484-46b9-88b0-18080838e296" />
<br><br>
A. 250,000 EGP
<br><br>

#### Q3: What is the name of the person to whom the victim owes money?
Found the address from the above screenshot (+201172137258). Investigated Contacts > Contacts.<br><br>
<img width="1277" height="793" alt="image" src="https://github.com/user-attachments/assets/35981c06-85e6-48a1-8f07-e271cac3bb26" />
<br><br>
A. Shady Wahab
<br><br>

### Step2: Understand victim's behavior
#### Q4: Based on the statement from the victim's family, they said that on September 20, 2023, he departed from his residence without informing anyone of his destination. Where was the victim located at that moment?
Recent Activity > Recent Activity 0, Found the snapshot image for Google Map.<br><br>
<img width="680" height="1446" alt="image" src="https://github.com/user-attachments/assets/78f80f8c-2739-4011-9dc1-1755c6202ee0" />
<br><br>

A. The Nile Ritz-Carlton
<br><br>
#### Q5: The detective continued his investigation by questioning the hotel lobby. She informed him that the victim had reserved the room for 10 days and had a flight scheduled thereafter. The investigator believes that the victim may have stored his ticket information on his phone. Look for where the victim intended to travel.
Google Photos > Google Photos (gphotos-1) - Cache, found the flight ticket.<br><br>
<img width="3793" height="1226" alt="image" src="https://github.com/user-attachments/assets/96b69289-0668-4c9b-a6a3-e208f42302f7" />
<br><br>
A. Las Vegas
<br><br>

#### Q6: After examining the victim's Discord conversations, we discovered he had arranged to meet a friend at a specific location. Can you determine where this meeting was supposed to occur?
Discord Chats > Discord Chats, found the message with the friend.<br><br>
<img width="1615" height="588" alt="image" src="https://github.com/user-attachments/assets/26fe3703-4449-4a5c-bf51-64f68632597b" />
<br><br>
A. The Mob Museum
<br><br>


## Key Findings / IOCs
| Type | Value |
|---|---|
| Trading App | Olymp Trade |
| App Hash | 4f168a772350f283a1c49e78c1548d7c2c6c05106d8b9feb825fdc3466e9df3c |
| Debt | 250,000 EGP |
| Lender | Shady Wahab |
| Location at Sep 20 | The Nile Ritz-Carlton |
| Travel Destination | Las Vegas |
| Meeting Location | The Mob Museum |

## Lessons Learned
- ALEAPP contains a lot of useful information, from installed apps, SMS, to search terms on Google. I sometimes hear the news about analyzing victim's or suspect's phones to extract information, but now understood a taste of how to do that.
- It's natural to consider that if our phones are compromized, attackers can see the same or above information, which reveals all of our lives literally. Maintaining secure environment around phones is so important.
