# Dancing
## Scope and Objective 
Target IP: 10.129.53.171  
Service: SMB  
Environment: HTB Pwnbox  

## About
Dancing is a very easy Windows machine which introduces the Server Message Block (SMB) protocol, its enumeration and its exploitation when misconfigured to allow access without a password.

| Lab | Platform | Difficulty | Focus |
| --- | --- | --- | --- |
| BFT | HackTheBox | Very Easy | DFIR |

### Task 1
What does the 3-letter acronym SMB stand for?  
SMB stands for Server Message Block

### Task 2
What port does SMB use to operate at?  
SMB uses TCP port `445`

### Task 3
What is the service name for port `445` that came up in our Nmap scan?  
To find the service name for Port `445`, we perform the following Nmap scan:
```bash
nmap -sV 10.129.53.171
```
The response of the scan is shown below:
```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-20 02:38 EDT
Nmap scan report for 10.129.53.171
Host is up (0.15s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.47 seconds
```
You can see that both port `445` and port `139` are open. SMB can operate on both ports, but in different ways. Port `139` is used for SMB over NetBIOS which is an older networking service that establishes and manages communication sessions between devices on a network. Port `445` is used by modern SMB to communicate directly over TCP/IP without requiring NetBIOS.  

Once we have discovered that SMB is running on the target, we can connect to the smbclient by running the following command:
```bash
smbclient -L 10.129.53.171
```
When prompted for a password, we simply leave it blank and hit `Enter` to tell the script to move along 
```bash
Password for [WORKGROUP\user]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk      
SMB1 disabled -- no workgroup available
```
As you can see, there are 4 different shares shown:
- `ADMIN$` with the comment `Remote Admin`, which could be interesting to investigate further
- `C$` is the C:\ directory, bascially where the OS is hosted
- `IPC$`
- `WorkShares` is a custom share
