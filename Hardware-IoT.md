# 🔌 Hardware & IoT Security Cheat Sheet

**Полное руководство по аппаратной безопасности и взлому IoT устройств.**  
Включает техники для UART, SPI, JTAG, анализа прошивок, извлечения данных и многое другое.

---

## 📚 Оглавление
1. [Базовая разведка устройства](#-базовая-разведка-устройства)
2. [UART (Universal Asynchronous Receiver-Transmitter)](#-uart)
3. [SPI (Serial Peripheral Interface)](#-spi)
4. [JTAG (Joint Test Action Group)](#-jtag)
5. [Анализ прошивок (Firmware Analysis)](#-анализ-прошивок-firmware-analysis)
6. [Извлечение данных с чипов памяти](#-извлечение-данных-с-чипов-памяти)
7. [Обратный инжиниринг прошивок](#-обратный-инжиниринг-прошивок)
8. [Атаки на IoT протоколы](#-атаки-на-iot-протоколы)
9. [Полезные инструменты](#-полезные-инструменты)

---

## 🔍 Базовая разведка устройства

### 1.1 Визуальный осмотр

```bash
# Что искать на плате:
# - UART контакты (TX, RX, GND, VCC)
# - SPI контакты (MISO, MOSI, SCK, CS)
# - JTAG контакты (TDI, TDO, TMS, TCK, TRST)
# - Тестовые точки (TP1, TP2, ...)
# - Маркировка чипов
# - Перемычки и джамперы

1.2 Идентификация интерфейсов
# Поиск контактов по цвету
# Черный - обычно GND
# Красный - VCC (3.3V или 5V)
# Белый/Желтый/Оранжевый - сигнальные линии

# Проверка напряжения мультиметром
# Установите режим измерения постоянного напряжения
# Черный щуп на предполагаемый GND
# Красный щуп на другие контакты

1.3 Фотодокументация
# Сфотографировать плату с обеих сторон
# Сделать макро-снимки чипов и контактов
# Зарисовать расположение контактов
# Отметить найденные интерфейсы

UART (Universal Asynchronous Receiver-Transmitter)
2.1 Обнаружение UART
# Признаки UART:
# - 4 контакта в ряд (GND, VCC, TX, RX)
# - 3 контакта (GND, TX, RX) - без питания
# - Контакты с маркировкой (J1, J2, CON1)

# Поиск с помощью осциллографа:
# - На TX всегда есть активность при загрузке
# - На RX обычно тишина или отклики
# - Скорость обычно 9600, 115200, 57600 бод

2.2 Подключение к UART
# Необходимое оборудование:
# - USB-to-UART адаптер (CP2102, FT232, CH340)
# - Перемычки (мама-мама)
# - Мультиметр для проверки напряжения

# Подключение:
# GND адаптера -> GND устройства
# TX адаптера -> RX устройства
# RX адаптера -> TX устройства

# Проверка в Linux
ls /dev/ttyUSB*
screen /dev/ttyUSB0 115200
# или
minicom -D /dev/ttyUSB0 -b 115200

2.3 Определение скорости UART
# Использование logic-анализатора
# PulseView (sigrok)
# Установка: sudo apt install sigrok pulseview

# Автоопределение скорости в Linux
stty -F /dev/ttyUSB0 115200
cat /dev/ttyUSB0

# Перебор скоростей (скрипт)
for baud in 9600 19200 38400 57600 115200 230400 460800; do
    echo "Testing $baud"
    stty -F /dev/ttyUSB0 $baud
    timeout 1 cat /dev/ttyUSB0
done

2.4 Эксплуатация UART доступа
# После подключения часто получаем:
# - Загрузочные логи
# - Shell (BusyBox, Linux)
# - U-Boot консоль
# - Recovery режим

# Команды в U-Boot
printenv
setenv bootargs console=ttyS0,115200
bootm

# Команды в BusyBox
ls
cat /etc/passwd
cat /etc/shadow
ps aux
mount
df -h

2.5 Прерывание загрузки
# Во время загрузки нажимаем:
# Enter, Space, Esc, Ctrl+C
# Часто срабатывает нажатие любой клавиши

# В U-Boot остановка обычно:
# Нажать любую клавишу в течение 3 секунд
# Или "Hit any key to stop autoboot"

# Изменение параметров загрузки
setenv bootargs console=ttyS0,115200 init=/bin/sh
boot

 SPI (Serial Peripheral Interface)
3.1 Обнаружение SPI
# Сигналы SPI:
# MISO (Master In Slave Out) - данные от устройства
# MOSI (Master Out Slave In) - данные к устройству
# SCK (Serial Clock) - тактовый сигнал
# CS (Chip Select) - выбор чипа

# Найти можно:
# - 4-6 контактов рядом
# - Маркировка: SPI, FLASH, JSPI
# - Контакты с резисторами подтяжки

3.2 Подключение к SPI
# Необходимое оборудование:
# - SPI программатор (Bus Pirate, FT2232H, CH341A)
# - SOIC-8 зажим или пайка

# Подключение через CH341A
# Монтаж SOIC-8 клипсы на микросхему памяти

# Чтение флеш-памяти
flashrom -p ch341a_spi -r firmware.bin

# Запись прошивки
flashrom -p ch341a_spi -w new_firmware.bin

# Проверка ID чипа
flashrom -p ch341a_spi --flash-size

3.3 Анализ SPI Flash
# Определение типа чипа
# Смотреть маркировку: 25Q64, 25L128, 25X10

# Установка необходимых пакетов
sudo apt install flashrom binwalk

# Чтение через Bus Pirate
flashrom -p buspirate_spi:dev=/dev/ttyUSB0 -r backup.bin

# Чтение через FT2232H
flashrom -p ft2232_spi:type=2232H -r firmware.bin

JTAG (Joint Test Action Group)
4.1 Обнаружение JTAG
# Сигналы JTAG:
# TDI (Test Data In) - данные на устройство
# TDO (Test Data Out) - данные от устройства
# TMS (Test Mode Select) - управление состоянием
# TCK (Test Clock) - тактовый сигнал
# TRST (Test Reset) - опционально

# Признаки:
# - 5-6 контактов в ряд
# - Маркировка: JTAG, DEBUG, J1, J2
# - Контакты с подтяжкой к питанию

4.2 Поиск JTAG контактов
# Метод перебора:
# 1. Найти GND (прозвонка с землей)
# 2. Найти VCC (3.3V)
# 3. Остальные проверять осциллографом

# Использование JTAGulator
git clone https://github.com/grandideastudio/jtagulator
cd JTAGulator
# Подключение к устройству
# Выбор режима обнаружения

4.3 Подключение к JTAG
# Оборудование:
# - JTAG программатор (J-Link, Bus Pirate, FT2232H)
# - OpenOCD для работы

# Установка OpenOCD
sudo apt install openocd

# Конфигурация OpenOCD
# openocd.cfg:
# source [find interface/ftdi/jlink.cfg]
# transport select jtag
# source [find target/stm32f1x.cfg]

# Запуск OpenOCD
openocd -f openocd.cfg

# Подключение telnet
telnet localhost 4444
# Команды OpenOCD
halt
reset
flash write_image firmware.bin 0x08000000
resume
exit

4.4 Эксплуатация JTAG
# Чтение памяти
dump_image memory.bin 0x08000000 0x10000

# Запись памяти
load_image firmware.bin 0x08000000

# Сброс и останов
reset halt

# Проверка регистров
reg

# Чтение флеш
flash read_bank 0 flash_dump.bin

Анализ прошивок (Firmware Analysis)
5.1 Извлечение прошивки
# Скачивание с сайта производителя
wget http://example.com/firmware.bin

# Извлечение через UART
# Во время загрузки захватить логи
screen -L /dev/ttyUSB0 115200

# Извлечение через SPI
flashrom -p ch341a_spi -r firmware.bin

# Извлечение через JTAG
openocd -f interface.cfg -f target.cfg -c "init; dump_image firmware.bin 0x00000000 0x100000; exit"

5.2 Первоначальный анализ
# Проверка типа файла
file firmware.bin
strings firmware.bin | head -50

# Поиск файловых систем
binwalk firmware.bin
binwalk -e firmware.bin

# Поиск сигнатур
binwalk -I firmware.bin
binwalk -A firmware.bin

5.3 Распаковка прошивки
# Распаковка squashfs
unsquashfs firmware.squashfs

# Монтирование ext2/3/4
mount -o loop firmware.ext2 /mnt/firmware

# Использование firmadyne
git clone https://github.com/firmadyne/firmadyne
cd firmadyne
./sources/extractor/extractor.py -b 5 -sqlite firmware.db firmware.bin

# Использование firmware-mod-kit
git clone https://github.com/rampageX/firmware-mod-kit
cd firmware-mod-kit
./extract-firmware.sh firmware.bin

5.4 Поиск уязвимостей
# Поиск строковых констант
strings firmware.bin | grep -i "password"
strings firmware.bin | grep -i "admin"
strings firmware.bin | grep -i "debug"

# Поиск hardcoded ключей
strings firmware.bin | grep -E "BEGIN (RSA|DSA|PRIVATE)"
strings firmware.bin | grep -E "[0-9a-f]{32}"

# Поиск уязвимых функций
grep -r "strcpy\|sprintf\|gets" extracted/
grep -r "system\|exec\|popen" extracted/

# Проверка версий библиотек
find extracted/ -name "*.so*" -exec strings {} \; | grep -i "version"

Извлечение данных с чипов памяти
6.1 Работа с EEPROM
# Чтение 24LCXX через i2c
i2cdetect -y 1
i2cget -y 1 0x50 0x00
i2cdump -y 1 0x50

# Использование flashrom
flashrom -p ch341a_spi -r eeprom.bin

# Специализированные программаторы
# - TL866
# - Xgecu
# - Minipro

6.2 Работа с NAND/NOR Flash
# Определение типа чипа
# NOR Flash - параллельный интерфейс, большие адресные линии
# NAND Flash - последовательный, страничная организация

# Чтение NOR через SPI программатор
flashrom -p ch341a_spi -r nor_flash.bin

# Чтение NAND через программатор
# Использование PC3000 Flash
# Использование Visual NAND Reconstructor

6.3 Восстановление данных
# Поиск файловых систем в дампе
binwalk -e firmware.bin

# Восстановление разделов
dd if=firmware.bin of=rootfs.bin bs=1024 skip=1234 count=5678

# Монтирование найденных разделов
mount -o loop,ro rootfs.bin /mnt/rootfs

# Анализ с помощью photorec
photorec /path/to/dump.bin

Обратный инжиниринг прошивок
7.1 Дисассемблирование
bash
# Использование objdump (Linux ELF)
objdump -d -M intel firmware.elf > disasm.txt

# Использование radare2
r2 firmware.bin
[0x00000000]> aaaa          # Анализ
[0x00000000]> afl            # Список функций
[0x00000000]> s main         # Переход к main
[0x00000000]> pdf            # Печать функции
[0x00000000]> VV             # Графический режим

# Использование Ghidra
# Загрузить firmware.bin
# Выбрать архитектуру (ARM, MIPS, x86)
# Анализ -> Автоматический анализ

7.2 Поиск строк и констант
# В radare2
[0x00000000]> iz             # Список строк
[0x00000000]> / "password"   # Поиск строки

# В Ghidra
# Search -> For Strings
# Search -> For Instructions

# Через командную строку
strings firmware.bin | grep -i "password" -A 5 -B 5

7.3 Отладка прошивки
# Использование QEMU для эмуляции
sudo apt install qemu-system-arm qemu-user-static

# Эмуляция ARM бинарника
qemu-arm-static -L extracted/lib extracted/bin/httpd

# Эмуляция MIPS бинарника
qemu-mips-static -L extracted/lib extracted/bin/httpd

# Эмуляция всей прошивки
# Использование firmadyne
./run.sh firmware_id

7.4 Патчинг прошивки
# Поиск места для патча
binwalk -e firmware.bin
cd _firmware.bin.extracted/

# Редактирование бинарника (hexedit)
hexedit squashfs-root/bin/httpd

# Изменение пароля в конфиге
vi squashfs-root/etc/passwd
# root:$1$...:0:0:root:/root:/bin/sh

# Перепаковка прошивки
./build-firmware.sh

Атаки на IoT протоколы
8.1 MQTT (Message Queuing Telemetry Transport)
# Поиск MQTT брокера
nmap -p 1883,8883 192.168.1.0/24

# Подключение к MQTT
mosquitto_sub -h 192.168.1.100 -p 1883 -t "#" -v
mosquitto_pub -h 192.168.1.100 -p 1883 -t "home/livingroom/light" -m "ON"

# Использование mqtt-pwn
git clone https://github.com/akamai-threat-research/mqtt-pwn
cd mqtt-pwn
python3 mqtt-pwn.py

8.2 CoAP (Constrained Application Protocol)
# Поиск CoAP серверов
nmap -p 5683 192.168.1.0/24

# Использование coap-client
coap-client -m get coap://192.168.1.100/.well-known/core
coap-client -m put -e "1" coap://192.168.1.100/light

# Анализ трафика
tcpdump -i eth0 port 5683 -w coap.pcap

8.3 ZigBee / Z-Wave
# Необходимое оборудование:
# - ATUSB (Atmel USB dongle)
# - CC2531 USB dongle

# Инструменты:
# - KillerBee (ZigBee)
# - EZ-Wave (Z-Wave)

# Перехват ZigBee пакетов
zbassocflood -i /dev/ttyUSB0

# Анализ Z-Wave
ezwave -d /dev/ttyUSB0

8.4 Bluetooth Low Energy (BLE)
# Поиск BLE устройств
sudo hcitool scan
sudo hcitool lescan
sudo bluetoothctl

# Использование gatttool
gatttool -b 00:11:22:33:44:55 -I
> connect
> primary
> characteristics
> char-read-hnd 0x0010

# Использование bettercap
sudo bettercap -eval "ble.recon on; ble.show"

Полезные инструменты
Инструмент	Назначение
Bus Pirate	Мультиинструмент для UART, SPI, JTAG, I2C
JTAGulator	Обнаружение JTAG контактов
CH341A	Дешевый программатор SPI Flash
FT2232H	Высокоскоростной JTAG/SPI программатор
Logic Analyzer	Анализ сигналов (Saleae, DSLogic)
flashrom	Чтение/запись Flash памяти
binwalk	Анализ и распаковка прошивок
radare2	Обратный инжиниринг
Ghidra	Дисассемблирование и анализ
QEMU	Эмуляция архитектур
firmadyne	Эмуляция IoT прошивок
OpenOCD	Работа с JTAG/SWD
sigrok/PulseView	Анализ логических сигналов
📚 Источники и дополнительная литература
Hardware All The Things

Firmware Analysis Toolkit

IoT Security Wiki

JTAG/SWD Pinout Database
