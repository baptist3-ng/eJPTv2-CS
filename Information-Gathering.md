# Summary
[What is it ?](#what-is-it-)

[Passive Information Gathering](#passive-information-gathering)

  - [Tools](#tools)
  - [DNS Recon](#dns-recon)
  - [WAF Analyse](#waf-analyse)
  - [Google Dorks](#google-dorks)
  - [Databases Leak](#databases-leak)

[Active Information Gathering](#active-information-gathering)

- [DNS Zone Transfer](#dns-zone-transfer)
- [Network Mapping](#network-mapping)
	- [Host Discovery](#host-discovery)
	- [Port Scanning](#port-scanning)
	- [Service Version and OS Detection](#service-version-and-os-detection)
	- [Firewall Detection & IDS Evasion](#firewall-detection--ids-evasion)


# What is it ?
---
The goal is to collect information about the target. This is the FIRST step of a pentest and one of the most important. There are two types of Information Gathering : 

- **Passive** Information Gathering
- **Active** Information Gathering

# Passive Information Gathering
---
Collect information with OSINT (Open-Source Intelligence).
Which type of Information is ? :

- IP Address
- Hidden directories from search engines
- Names
- Email
- Phone Number
- Address (physical)
- Web technologies used


## Tools

- Whois : DNS Information
- **Whatweb** : **CMS** Identifier
- Netcraft : Website for Passive Information Gathering
- **Sublist3r** : Passive subdomain enumeration
- **theHarvester** : Collect info about subdomain, email, ...

## DNS Recon

Enumerate information about **DNS** server.

- **dnsrecon** on Kali Linux : 

```bash
dnsrecon -d https://zonetransfer.me
```


- [DNSDumpster](https://dnsdumpster.com/)

	Go to website and enter the target URL. The *View Graph* is useful !


## WAF Analyse

Sometimes, the website can be protected by a Web Application Firewall (WAF).

- **WafW00f** : 
```bash
wafw00f https://zonetransfer.me
```

Use `-a` to check all possibilities ! 

## Google Dorks

Use Google to get more info about the target thanks to dorks. 

- Search for subdomains : 

```txt
site:*.website.com
```

- Search for specific words : 

```txt
inurl:password
intitle:admin
```

More dorks on [GHDB](https://www.exploit-db.com/google-hacking-database).

## Databases Leak

Sometimes, an email can be found in some data breach. You can use [HaveIBeenPnwed](https://haveibeenpwned.com/) to check.

If you found leaked passwords, you can do a *password spraying* attack.


# Active Information Gathering
---
In this phase, we are now engaging with the target's machine. The goal is the same as the previous section.

## DNS Zone Transfer

It involves an attacker requesting and receiving a copy of the entire **DNS** database, also known as a zone file, from a **DNS** server.

- dnsEnum : 

```bash
dnsenum zonetransfer.me
```

- dig : 

```bash
dig axfr @NAMESERVER zonetransfer.me
```

- fierce : 

```bash
fierce -dns zonetransfer.me
```

*This tool can also brute-force subdomains*.


## Network Mapping

>Network mapping in the context of penetration testing refers to the process of discovering and identifying devices, hosts, and network infrastructure elements within a target network.

### Host Discovery

We can perform a pingsweeps by sending ICMP packets to each host of the range and wait for a response.

```bash
fping -g 172.31.0.0/24
```

We can also use **Nmap** : 
```bash
nmap -sn 192.168.1.0/24
```

With a target file : 
```bash
nmap -sn -iL targets.txt
```

Using SYN Ping scan (really useful !) : 
```bash
nmap -sn -PS 192.168.1.27
```

### Port Scanning

Simple scan with **Nmap** : 
```bash
nmap 10.10.10.1
```

Stealth Scan (RECOMMENDED) : 
```bash
nmap -sS 10.10.10.1
```

***Important*** : If you are root, **Nmap** will perform a SYN Scan. If not, it will perform a TCP Scan. The difference is that SYN Scan doesn't create a TCP Connection (3 Way-Handshake).

**UDP** Scan : 
```bash
nmap -sU 10.10.10.1
```

Custom port range : 
```bash
nmap -p- 10.10.10.1
nmap -p1-5000 10.10.10.1
nmap -p80,443,3389,8080 10.10.10.1
```


### Service Version and OS Detection

Get version of service : 
```bash
nmap -sV 10.10.10.1 -p8080,443
```

Get OS version : 
```bash
nmap -O 10.10.10.1
```

You can also use *aggressive* scan : 
```bash
nmap -A 10.10.10.1 -p80,8080,443
```


### Firewall Detection & IDS Evasion

Send fragmented packets : 
```bash
nmap -sS -p80,445,3389 -f --mtu 8 10.10.10.1
```

IP Address spoofing : 
```bash
nmap -sS -p80,445,3389 -f --mtu 8 -D 192.168.1.1 --data-length 200 192.168.1.24
```

Change source port with `-g` :
```bash
nmap -sS -p80,445,3389 -f --mtu 8 -g 53 -D 192.168.1.1 --data-length 200 192.168.1.24
```

Increase time delay (less suspicious): 
```bash
nmap -sS -p80,3389,445 -sV --scan-delay 15s 192.168.1.24
```

Same with `-T` option : 
```bash
nmap -sS -p80,3389,445 -sV -T1 192.168.1.24
```

