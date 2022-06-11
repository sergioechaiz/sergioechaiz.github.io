---
layout: post
title:  "pruebaassss"
date:   2022-03-16
excerpt: "Custom writte."
project: true
tag:
- sample
- post
- video
comments: true
---
~~~~~~~~
#!/bin/bash

if [ -z "$1" ]
then
        echo -e  "\e[33m[*]-Use: ./nmapScan\e[0m" "\e[31m[<IP>]\e[0m"
        exit 1
fi

printf "\n[*] Puertos Abiertos:\n\n" > OpenPorts

echo -e "\e[31m Escaneando Puertos Abiertos...\e[0m"

sudo nmap -sS -p- --open --min-rate 5000 $1 -n -Pn | tail -n +5 | head -n -3 >> OpenPorts

cat OpenPorts
