# hw08
**1. Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig**

*Установил для работы новую виртуальную машину centos/8*

*Создал файл с конфигурацией для сервиса в директории /etc/sysconfig - из неё сервис будет брать необходимые переменные.*

[root@localhost ~]# vi /etc/sysconfig/watchlog

\# Configuration file for my watchdog service

\# Place it to /etc/sysconfig

\# File and word in that file that we will be monit

WORD="ALERT"

LOG=/var/log/watchlog.log

*Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение, плюс ключевое слово ‘ALERT’*

*Создаю скрипт*

[root@localhost ~]# vi /opt/watchlog.sh

\#!/bin/bash

WORD=$1

LOG=$2

DATE=`date`

if grep $WORD $LOG &> /dev/null

then

logger "$DATE: I found word, Master!"

else

exit 0

fi

*Сделал скрипт исполняемым*


[root@localhost ~]# chmod u+x /opt/watchlog.sh

*Создал Юнит для сервиса:*
[root@localhost ~]# vi /etc/systemd/system/watchlog.service

[Unit]

Description=My watchlog service

[Service]

Type=oneshot

EnvironmentFile=/etc/sysconfig/watchdog

ExecStart=/opt/watchlog.sh $WORD $LOG

*Создал Юнит для запуска каждые 30 секунд*

[root@localhost ~]# vi /etc/systemd/system/watchlog.timer

[Unit]

Description=Run watchlog script every 30 second

[Timer]

\# Run every 30 second

OnUnitActiveSec=30

Unit=watchlog.service

[Install]

WantedBy=multi-user.target

**Стартую таймер**

[root@localhost ~]# systemctl start watchlog.timer

** Проверяю результат** 

[root@localhost ~]# tail -f /var/log/messages
Dec 13 07:12:23 localhost NetworkManager[643]: <info>  [1639379543.5772] device                                                   (eth1): state change: config -> ip-config (reason 'none', sys-iface-state: 'mana                                                  ged')
Dec 13 07:12:23 localhost NetworkManager[643]: <info>  [1639379543.5781] dhcp4 (                                                  eth1): activation: beginning transaction (timeout in 45 seconds)
Dec 13 07:13:08 localhost NetworkManager[643]: <warn>  [1639379588.5272] dhcp4 (                                                  eth1): request timed out
Dec 13 07:13:08 localhost NetworkManager[643]: <info>  [1639379588.5274] dhcp4 (                                                  eth1): state changed unknown -> timeout
Dec 13 07:13:08 localhost NetworkManager[643]: <info>  [1639379588.5275] device                                                   (eth1): state change: ip-config -> failed (reason 'ip-config-unavailable', sys-i                                                  face-state: 'managed')
Dec 13 07:13:08 localhost NetworkManager[643]: <warn>  [1639379588.5301] device                                                   (eth1): Activation: failed for connection 'Wired connection 1'
Dec 13 07:13:08 localhost NetworkManager[643]: <info>  [1639379588.5309] device                                                   (eth1): state change: failed -> disconnected (reason 'none', sys-iface-state: 'm                                                  anaged')
Dec 13 07:13:08 localhost NetworkManager[643]: <info>  [1639379588.5600] dhcp4 (                                                  eth1): canceled DHCP transaction
Dec 13 07:13:08 localhost NetworkManager[643]: <info>  [1639379588.5601] dhcp4 (                                                  eth1): state changed timeout -> done
Dec 13 07:14:50 localhost systemd[1]: Started Run watchlog script every 30 secon                                                  d.

  
