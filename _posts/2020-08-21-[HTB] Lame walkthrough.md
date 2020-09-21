---
layout: post
title:  "[HTB] Lame walkthrough"
tags: htb walktrough pentesting
---

# Intro

Pierwsze walktrough dość już starej maszynki o wdzięcznej nazwie "Lame" na platformie [Hack the box](https://www.hackthebox.eu/). Jeśli nie wiesz czym jest platforma HTB to spieszę z wyjaśnieniem. Platforma HTB to w dużym skrócie serwis, który umożliwia testowanie i rozwijanie umiejętności z zakresu pentestingu. Zabawy polega na zdobyciu dwóch flag: zwykłego użytkownika oraz roota (Administratora dla systemów Windows). Opis całej platformy to temat na oddzielny post. Zatem przejdźmy do "shackowania" naszej pierwszej maszynki.

# Rekonesans

W dalszej części postus przyjąłem, że posiadasz skonfigurowane połączenie VPN do platformy HTB. 

Zawsze pierwszym krokiem do zdobycia dostępu do maszyny jest rekonesans. Musimy sprawdzić jakie porty są otwarte oraz jakie usługi zostały wystawione. W tym celu użyjemy najpopularniejszego narzędzia do skanowania portów jakim jest nmap. Otwieramy konsole i wydajemy następujące polecenie:

```bash
sudo nmap -sVC 10.10.10.3
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-21 18:08 UTC
Stats: 0:00:18 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.88% done; ETC: 18:08 (0:00:00 remaining)
Nmap scan report for 10.10.10.3
Host is up (0.020s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.36
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -3d00h50m32s, deviation: 2h49m45s, median: -3d02h50m35s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2020-09-18T11:18:20-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: userhiw
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.54 seconds

```

Krótkie wyjaśnienie przełączników:
* **-sC** - skanowanie z domyślnym zestawem skryptów
* **-V** - pokaż wersję usług

Jak widzimy na powyższym zrzucie mamy kilka usług: ftp, ssh oraz sambę. SSH raczej możemy odrzucić, rzadko jest to punkt wejścia do maszyny, chyba, że innym sposobem (np. poprzez wykorzystanie dziury w aplikacji webowej) udało się zdobyć poświadczenia do logowania. Zostaje zatem ftp oraz samba. Szybkie wyszukanie w google frazy "vsftpd 2.3.4" prowadzi nas do strony (https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor). Według opisu tej podatności do archiwum vsftp został wprowadzony backdoor, który pozwala na wykonanie dowolnego polecenia na systemie z uprawnieniami usługi vsftpd. Czyli mamy podatność typu RCE (Remote Code Execution). Możemy zatem przejść do próby eksploitacji. 

# Eksploitacja wersja łatwa

W pierwszej wersji wykorzystamy narzędzie metasploit, które zawiera gotowy do wykorzystania eksploit. Uruchamiamy metasploita:

```bash
msfconsole
                                                  
 _                                                    _
/ \    /\         __                         _   __  /_/ __
| |\  / | _____   \ \           ___   _____ | | /  \ _   \ \
| | \/| | | ___\ |- -|   /\    / __\ | -__/ | || | || | |- -|
|_|   | | | _|__  | |_  / -\ __\ \   | |    | | \__/| |  | |_
      |/  |____/  \___\/ /\ \\___/   \/     \__|    |_\  \___\


       =[ metasploit v5.0.88-dev                          ]
+ -- --=[ 2014 exploits - 1097 auxiliary - 343 post       ]
+ -- --=[ 562 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: Enable verbose logging with set VERBOSE true

msf5 > 
```

# Eksploitacja wersja pro - piszemy eksploita w Pythonie
# Podsumowanie
