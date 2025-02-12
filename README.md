# EDR-Attack-and-Defense

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
















