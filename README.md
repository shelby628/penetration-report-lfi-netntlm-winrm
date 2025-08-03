# penetration-report-lfi-netntlm-winrm
# LFI to WinRM Access via NetNTLMv2 Exploitation

This report documents a full attack chain starting from a Local File Inclusion (LFI) vulnerability leading to full system access using captured NetNTLMv2 hashes and Evil-WinRM.

Author: Shelby Adede  
Target: HTB Machine  
Date: 2nd August 2025  

 Summary

- Web Enumeration revealed an LFI vulnerability
- Forced SMB authentication triggered with a remote path
- NetNTLMv2 hash captured using Responder
- Credentials cracked using John the Ripper
- Remote shell obtained using Evil-WinRM

This lab demonstrates a realistic end-to-end penetration testing scenario, including enumeration, exploitation, hash cracking, and privilege escalation.
