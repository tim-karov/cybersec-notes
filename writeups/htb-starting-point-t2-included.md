# HTB Starting Point Tier 2: Included

- **Difficulty:** Very Easy
- **Дата:** 12.01.2026
- **Цель:** Практика базового enumeration и простого эксплойта Log4Shell.

## 1. Recon & Enumeration
- Nmap, UDP сканирование, запускаем с `-sU`, и можно чуть шумнее и быстрее с `-T4`, ибо UDP сканирование работает значительно дольше.: `sudo nmap -sU -sC -sV -v -T4 $ip`. Находим открытый порт `69`, на котором запущен `TFTP`. `TFTP` – упрощенный протокол передачи файлов, предназначенный для быстрой и простой передачи небольших файлов.

## 2. Exploitation


## 3. Privilege Escalation


## Done
