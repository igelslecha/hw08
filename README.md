# hw08
**1. Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig**
*Установил для работы новую виртуальную машину centos/8*
*Создал файл с конфигурацией для сервиса в директории /etc/sysconfig - из неё сервис будет братþ необходимые переменные.*
[root@localhost ~]# vi /etc/sysconfig/watchlog
* # Configuration file for my watchdog service*
*# Place it to /etc/sysconfig*
*# File and word in that file that we will be monit*
WORD="ALaRm"
LOG=/var/log/watchlog.log

*Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение, плюс ключевое слово ‘ALaRm’*

*Создаю скрипт*

[root@localhost ~]# vi /opt/watchlog.sh
#!/bin/bash
WORD=$1
LOG=$2
DATE=`date`
if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi


