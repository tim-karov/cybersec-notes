# THM: "Mustacchio" Machine Walkthrough

- **Difficulty:** Easy / 
- **Дата:** 24.01.2026

## 1. Recon & Enumeration
Nmap `nmap -sC -sV -v $ip`, получаем 2 открытых порта `22 (SSH)`, `80 (HTTP)`. Переходим на веб-сайт, который расположен на 80м порте и, в общем и целом, ничего пригодного не находим, начинам фазинг каталогов: `ffuf -u http://machine_ip/FUZZ -w /usr/share/wordlists/dirb/common.txt`, находим интересный эндпоинт: `/custom`, перейдя по нему и в папке `js` находим `user.bak`. Читаем файл находим креды админа: `admin:1868e36a6d2b17d4c2745f1659433a54d4bc5f4b`. На первый взгляд хеш-пароля несложный, пробуем извлечь пароль в сервисе [Hashes.com](https://hashes.com/en/decrypt/hash), и очень быстро получаем пароль: `bulldog19`. И теперь возникает вопрос, а где админка, вряд-ли же это SSH-креды. Возвращаемся к nmap'у и пробуем просканить большее количество портов: `nmap -sC -sV -v -p- $ip`, находим порт `8765`, переходим к нему в браузере и попадаем на страницу авторизации панели админа, где и пригодились наши креды, заходим. Изучив исходный код страницы, в секции script нашел эндопинт: `/auth/dontforget.bak`, который загружает пример XML-разметки комментария с некоторыми полями. Также в исходном коде старницы есть интересный комментарий: `Barry, you can now SSH in using your key!`. Предполагаю здесь может быть XXE уязвимость. На Portswigger есть статья на этот счет — [What is XXE?](https://portswigger.net/web-security/xxe).

## 2. Exploitation
В теории, если риложение не имеет средств защиты от XXE-атак, мы можетм использовать уязвимость XXE для чтения файлов. Попробуем использовать следующую полезную нагрузку, чтобы прочитать файл `/etc/passwd`:
```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>&xxe;</com>
</comment>
```
Успешно, файл прочитан. Опираясь на комментарий в исходном коде, нам нужно дойти таким образом до SSH-ключа пользователя `Barry`, пробуем следующий путь: `home/barry/.ssh/id_rsa` и получаем приватный ключ, который мы записываем в отдельный файл, в моем примере это `privkey`, даем этому файлу полный права доступа `sudo chmod 700 privkey` и подключаемся: `sudo ssh -i private barry@machine_ip`, но встречаем в лицо это: `Enter passphrase for key 'private':`. Окей, тулзой ssh2john получим хеш ключа: `sudo ssh2john private > hash`, и теперь пробуем извлечь хеш с помощью словаря `rockyou.txt`: `john hash --wordlist=/usr/share/wordlists/rockyou.txt`. Получаем passphrase: `urieljames` и коннектимся используя ее. Успех, сразу же забираем user-флаг и приступаем к поиску веторов для повышения привилегий.

## 3. Privilege Escalation

## Done
