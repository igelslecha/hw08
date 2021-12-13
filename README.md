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

[root@localhost ~]# vi /var/log/watchlog.log

Hunter had seen alien and after louded ALERT

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
Dec 13 07:37:08 localhost NetworkManager[643]: <info>  [1639381028.5275] device (eth1): state change: ip-config -> failed (reason 'ip-config-unavailable', sys-iface-state: 'managed')
Dec 13 07:37:08 localhost NetworkManager[643]: <warn>  [1639381028.5301] device (eth1): Activation: failed for connection 'Wired connection 1'
Dec 13 07:37:08 localhost NetworkManager[643]: <info>  [1639381028.5307] device (eth1): state change: failed -> disconnected (reason 'none', sys-iface-state: 'managed')
Dec 13 07:37:08 localhost NetworkManager[643]: <info>  [1639381028.5595] dhcp4 (eth1): canceled DHCP transaction
Dec 13 07:37:08 localhost NetworkManager[643]: <info>  [1639381028.5595] dhcp4 (eth1): state changed timeout -> done
Dec 13 07:37:42 localhost systemd[1]: watchlog.timer: Succeeded.
Dec 13 07:37:42 localhost systemd[1]: Stopped Run watchlog script every 30 second.
Dec 13 07:37:42 localhost systemd[1]: Stopping Run watchlog script every 30 second.
Dec 13 07:37:42 localhost systemd[1]: Started Run watchlog script every 30 second.
Dec 13 07:40:29 localhost vagrant[3043]: Mon Dec 13 07:40:29 UTC 2021: I found word, Master!

**2. Из репозитория epel установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).**
  
  * Устанавливаю spawn-fcgi и необходимые для него пакеты:*
  
[root@localhost ~]# yum install epel-release -y && yum install spawn-fcgi php php-cli

 
Installed:
  apr-1.6.3-12.el8.x86_64                                            apr-util-1.6.1-6.el8.x86_64
  apr-util-bdb-1.6.1-6.el8.x86_64                                    apr-util-openssl-1.6.1-6.el8.x86_64
  centos-logos-httpd-85.8-2.el8.noarch                               httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64
  httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch     httpd-tools-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64
  mailcap-2.1.48-3.el8.noarch                                        mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64
  nginx-filesystem-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch     php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-cli-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                php-common-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
  php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64                spawn-fcgi-1.6.3-17.el8.x86_64

Complete!

  *Раскомментирую строки с переменными в /etc/sysconfig/spawn-fcgi*
  
\# You must set some working options before the "spawn-fcgi" service will work.

  \# If SOCKET points to a file, then this file is cleaned up by the init script.

  \#

  \# See spawn-fcgi(1) for all possible options.

  \#

  \# Example :

  SOCKET=/var/run/php-fcgi.sock

  OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

  *Создаю Юнит сервис*
  
  [root@localhost ~]# vi /etc/systemd/system/spawn-fcgi.service

[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target
[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process
[Install]
WantedBy=multi-user.target

*Запускаю*
  
  [root@localhost ~]# systemctl start spawn-fcgi

  *убеждаюсь, что сервис стартанул*
  
   spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-12-13 08:04:13 UTC; 56s ago
 Main PID: 22223 (php-cgi)
    Tasks: 33 (limit: 1133)
   Memory: 28.0M
   CGroup: /system.slice/spawn-fcgi.service
           ├─22223 /usr/bin/php-cgi
           ├─22224 /usr/bin/php-cgi
           ├─22225 /usr/bin/php-cgi
           ├─22226 /usr/bin/php-cgi
           ├─22227 /usr/bin/php-cgi
           ├─22228 /usr/bin/php-cgi
           ├─22229 /usr/bin/php-cgi
           ├─22230 /usr/bin/php-cgi
           ├─22231 /usr/bin/php-cgi
           ├─22232 /usr/bin/php-cgi
           ├─22233 /usr/bin/php-cgi
           ├─22234 /usr/bin/php-cgi
           ├─22235 /usr/bin/php-cgi
           ├─22236 /usr/bin/php-cgi
           ├─22237 /usr/bin/php-cgi
           ├─22238 /usr/bin/php-cgi
           ├─22239 /usr/bin/php-cgi
           ├─22240 /usr/bin/php-cgi
           ├─22241 /usr/bin/php-cgi
           ├─22242 /usr/bin/php-cgi
           ├─22243 /usr/bin/php-cgi
           ├─22244 /usr/bin/php-cgi
           ├─22245 /usr/bin/php-cgi
           ├─22246 /usr/bin/php-cgi
           ├─22247 /usr/bin/php-cgi
           ├─22248 /usr/bin/php-cgi
           ├─22249 /usr/bin/php-cgi
           ├─22250 /usr/bin/php-cgi
           ├─22251 /usr/bin/php-cgi
           ├─22252 /usr/bin/php-cgi
           ├─22253 /usr/bin/php-cgi
           ├─22254 /usr/bin/php-cgi
           └─22255 /usr/bin/php-cgi

Dec 13 08:04:13 localhost.localdomain systemd[1]: Started Spawn-fcgi startup service by Otus.

  **3. Дополнить Юнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигами**
  
  *создаю файл юнита*
  
  [root@localhost ~]# vi /etc/systemd/system/httpd.service

  [Unit]

  Description=The Apache HTTP Server

  After=network.target remote-fs.target nss-lookup.target

  Documentation=man:httpd(8)

  Documentation=man:apachectl(8)

  [Service]

  Type=notify

  EnvironmentFile=/etc/sysconfig/httpd-%I

  ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND

  ExecReload=/usr/sbin/httpd $OPTIONS -k graceful

  ExecStop=/bin/kill -WINCH ${MAINPID}

  KillSignal=SIGCONT

  PrivateTmp=true

  [Install]

  WantedBy=multi-user.target

  *Задаю опции для запуска веб-сервера с необходимым конфигурационным файлом*
  
  [root@localhost ~]# vi /etc/sysconfig/httpd-first
  
  \# /etc/sysconfig/httpd-first
  
  OPTIONS=-f conf/first.conf

  [root@localhost ~]# vi /etc/sysconfig/httpd-second

  \# /etc/sysconfig/httpd-second
  
  OPTIONS=-f conf/second.conf
  
  *Соответственно в директории с конфигами httpd создаю два конфига, first.conf и second.conf*
  
  [root@localhost ~]# mv /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf

  [root@localhost ~]# cp /etc/httpd/conf/first.conf /etc/httpd/conf/second.conf

  [root@localhost ~]# vi /etc/httpd/conf/second.conf

  *Запускаю и проверяю порты*
  
  [root@localhost ~]# systemctl start httpd@first

  [root@localhost ~]#  systemctl start httpd@second

  [root@localhost ~]# ss -tnulp | grep httpd

  tcp     LISTEN   0        128                    *:8080                *:*       users:(("httpd",pid=22556,fd=4),("httpd",pid=22555,fd=4),("httpd",pid=22554,fd=4),("httpd",pid=22551,fd=4))
tcp     LISTEN   0        128                    *:80                  *:*       users:(("httpd",pid=22333,fd=4),("httpd",pid=22332,fd=4),("httpd",pid=22331,fd=4),("httpd",pid=22328,fd=4))

  [root@localhost ~]#
