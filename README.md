# LimaCharlie-Attack/Defense

## Objective
This lab is designed to replicate a real-world cyber attack scenario while evaluating endpoint detection and response (EDR) capabilities. By following Eric Capuano's online guide, I will configure virtual machines to represent both the attacking and victim systems. The attack machine will leverage the 'Sliver' command and control (C2) framework to conduct an attack against a Windows endpoint. To counteract and analyze the attack, the Windows machine will be equipped with 'LimaCharlie' as its EDR solution. This setup will allow for a hands-on exploration of cyber threats, detection techniques, and response strategies in a controlled environment.

Eric Capuano's Guide: https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro?utm_campaign=post&utm_medium=web

### Skills Learned
- Simulating attacks in a controlled lab environment
- Learning how EDR solutions (e.g., LimaCharlie) detect and respond to threats
- Configuring EDR tools for threat detection and analysis
- Working with Sliver, an open-source C2 framework for red teaming
- Basic understanding how attackers use C2 channels to maintain access
- Analyzing system logs for signs of compromise
- Investigating cybersecurity incidents and mitigating threats


### Tools Used
- VMware
- Sliver
- LimaCharlie
- Sysmon

## Installation & Deployment
Download & deploy VMware Workstation Pro, Windows VM, and Ubuntu VM

<img width="133" alt="Screenshot 2025-02-11 195544" src="https://github.com/user-attachments/assets/e7c6ed36-54d7-4ddf-b920-11ce5a58704b" />

Install Sysmon in Window VM (must have analyst tool)

![Screenshot 2025-02-11 180017](https://github.com/user-attachments/assets/fc3cc043-fbd0-44de-963d-78a7d7726e3a)

Install LimaCharlie EDR on Window VM
- Install sensors and configure LimaCharlie to ship out Sysmon event logs + Sigma rule logs

![Screenshot 2025-02-11 180254](https://github.com/user-attachments/assets/38c91d32-aef4-4789-88ef-5cbd0cb4b35b)

![Screenshot 2025-02-11 180237](https://github.com/user-attachments/assets/6c8aaf12-29db-485b-8b8f-a11d88cd6c34)

## Generate Attack with Sliver
Disable Window Defender on Window VM

![Screenshot 2025-02-11 175813](https://github.com/user-attachments/assets/33a51cf9-20d4-4812-8a39-1d82dc13a6c5)

Generate & Implant C2 payload with Sliver

<img width="308" alt="Screenshot 2025-02-12 112300" src="https://github.com/user-attachments/assets/302359e2-bde0-43d5-bd95-a9513935339e" />

Spin up a temporary web server to download C2 payload onto the window VM

python3-m http.server 80

Start command and control session with Sliver server and execute C2 payload on Window VM

<img width="740" alt="Screenshot 2025-02-12 151526" src="https://github.com/user-attachments/assets/c0b9968a-487d-48a8-9c1a-09a2a77dabab" />

"<img width="259" alt="Screenshot 2025-02-12 152153" src="https://github.com/user-attachments/assets/3bc511a0-2a3c-43f4-a8ff-27db38fbe690" />"

Run series of commands to get more info from the victim's host (info, whoami, getprivs, pwd, netsta, ps-T)
- Sliver detects security products a victim may be using with the color red and detects their own processes in green
<img width="208" alt="Screenshot 2025-02-12 153813 (2)" src="https://github.com/user-attachments/assets/1820de98-ffa6-4aba-ac1b-924beed1e82f" />
<img width="202" alt="Screenshot 2025-02-12 153845 (2)" src="https://github.com/user-attachments/assets/4a21d429-a91b-4441-8899-5b05e63b978a" />

## Detect with LimaCharlie
Look for unsigned processes (but becareful of LOLBINs)
- STRANGE_SAMPAN.exe has powershell.exe as the parent process (powershell.exe has explorer.exe as the parent process)
- Having explorer.exe as the parent process lets us know this process happened in user mode (ran from the desktop)
- Also see the source ip & port number and destination ip & port number

![2025-02-12 12_05_03-hideonbush - window-vm localdomain - Processes - LimaCharlie](https://github.com/user-attachments/assets/ec7ba52f-e6da-4dcb-8f4e-662164a9250d)
![2025-02-12 11_38_19-hideonbush - window-vm localdomain - Processes - LimaCharlie](https://github.com/user-attachments/assets/06f6dd49-b2da-43d3-80d7-b92a5513ce6e)

The network section indicates that this process attempted to establish multiple network connections to our Windows VM.
- Need to investigate unusual processes using port 80 when browser should be the ones using port 80
- Hash has been created through this process --> VirusTotal
- Item not found in VirusTotal does not mean the file is innocent --> should be more curious on why

![2025-02-12 11_39_39-hideonbush - window-vm localdomain - Network - LimaCharlie](https://github.com/user-attachments/assets/4fdb52c6-33b0-4824-8f3f-6b198c523733)
![2025-02-12 11_40_47-hideonbush - window-vm localdomain - Network - LimaCharlie](https://github.com/user-attachments/assets/767611e2-ce2d-48d8-9681-b616183ad242)
![2025-02-12 11_42_22-VirusTotal - Search - 9298624932b6d68e3f334875db94545edfc0b4fa6237266cb95d5405a1](https://github.com/user-attachments/assets/a64cc5b9-03ba-4aea-8712-6e29481a12cb)

Utilize the timeline to track when the process was created and when it established network connections
![2025-02-12 16_03_07-hideonbush - window-vm localdomain - Timeline - LimaCharlie](https://github.com/user-attachments/assets/6077fd3e-c5b4-4efd-a6bb-446dcec99b8a)

## Perform Memory Dump within Sliver Session and Build a Detection Rule
![2025-02-12 16_23_02-So you want to be a SOC Analyst_ Part 3 - by Eric Capuano](https://github.com/user-attachments/assets/fe78736d-c751-494a-a274-33452cdd4542)
- Adversaries likes to target lsass.exe for credential dump because it stores user and domain admin credentials
- Build a detection rule(D&R) from the detected events in timeline
- RULE: Focus only on sensitive events where the victim or target process ends with lsass.exe, and generate a report whenever this detection happens.
  
![2025-02-12 16_26_11-blwhit_EDR-Attack-and-Defense_ Cyber Attack_Defense home lab using Sliver, LimaC](https://github.com/user-attachments/assets/bb299635-d6da-47f9-b39c-cdbe287f2a9a)

## Let's now Block Attacks instead of Generating Alerts
Started a remote shell within the Sliver session to run the command (vssadmin delete shadowns /all)
- attractive options for adversaries because it will delete all recovery files and file system (good for ransomware)
- still have an active shell after this command

<img width="551" alt="Screenshot 2025-02-12 190600" src="https://github.com/user-attachments/assets/996d3009-7e6a-453c-a4f5-757017a3fda4" />

LimaCharlie detected that activity with their default Sigma Rules
![2025-02-12 16_35_52-](https://github.com/user-attachments/assets/cbeaa68c-bb11-4bd0-85b3-c6447aa1457d)
![2025-02-12 16_36_05-hideonbush - window-vm localdomain - Timeline - LimaCharlie](https://github.com/user-attachments/assets/d090ccd4-ecad-4cb5-bb04-337ceb308d21)

View event timeline and craft a D&R rule for this event
- action: report (this section simply write a report to the detection tab)
- action:task (this section is what is reponsible for killing the parent process responsible with deny tree the process command

![2025-02-12 16_37_49-hideonbush - window-vm localdomain - Timeline - LimaCharlie](https://github.com/user-attachments/assets/6afcd735-badd-4a16-beac-c7549f0c3b79)

After rerunning the same command, D&R has successfully terminated the parent process 
- Remote shell fails to return anything
<img width="291" alt="Screenshot 2025-02-12 190748" src="https://github.com/user-attachments/assets/01b5e5dd-4b70-4d6f-88cb-b5ecdac57b4c" />



















