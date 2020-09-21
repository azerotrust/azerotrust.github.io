---
layout: post
title:  "[HTB] Lame walkthrough"
tags: htb walktrough pentesting
---

# Intro

Pierwsze walktrough dość już starej maszynki o wdzięcznej nazwie "Lame" na platformie [Hack the box](https://www.hackthebox.eu/). Jeśli nie wiesz czym jest platforma HTB to spieszę z wyjaśnieniem. Platforma HTB to w dużym skrócie serwis, który umożliwia testowanie i rozwijanie umiejętności z zakresu pentestingu. 

# Rekonesans

Zawsze pierwszym krokiem do zdobycia dostępu do maszyny jest rekonesans. Musimy sprawdzić jakie porty są otwarte oraz jakie usługi zostały wystawione. W tym celu użyjemy najpopularniejszego narzędzia do skanowania portów jakim jest nmap. Otwieramy konsole i wydajemy następujące poleceni:

```bash
nmap -sVC 10.10.10.3
```
