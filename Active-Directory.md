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

# PowerView (входит в PowerSploit)
Get-NetUser | select samaccountname, description
Get-NetComputer | select dnshostname, operatingsystem
