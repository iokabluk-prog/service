# Создаем файл с конфигурациуй для сервиса
ubuntuadmin@ubuntu01:~$ cat /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
# Создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение,
плюс ключевое слово ‘ALERT’
ubuntuadmin@ubuntu01:~$ cat /var/log/watchlog.log
test
test
test
ALERT
test
1
1
1
2
2
2
3
3
33
# Создаем скрипт
ubuntuadmin@ubuntu01:~$ cat /opt/watchlog.sh
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
# Добавляем права для запуска
ubuntuadmin@ubuntu01:~$ sudo chmod +x /opt/watchlog.sh
# Создадим юнит для сервиса
ubuntuadmin@ubuntu01:~$ sudo cat  /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
# Создадим юнит для таймера
ubuntuadmin@ubuntu01:~$ cat /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
# Запускаем timer
ubuntuadmin@ubuntu01:~$ systemctl start watchlog.timer
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to start 'watchlog.timer'.
Authenticating as: admin (ubuntuadmin)
Password:
==== AUTHENTICATION COMPLETE ====
Warning: The unit file, source configuration file or drop-ins of watchlog.timer changed on disk. Run 'systemctl daemon-reload' to reload units.
ubuntuadmin@ubuntu01:~$ systemctl daemon-reload
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
Authentication is required to reload the systemd state.
Authenticating as: admin (ubuntuadmin)
Password:
==== AUTHENTICATION COMPLETE ====
ubuntuadmin@ubuntu01:~$ sudo systemctl start watchlog.timer
ubuntuadmin@ubuntu01:~$ sudo systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
     Loaded: loaded (/etc/systemd/system/watchlog.timer; disabled; preset: enabled)
     Active: active (elapsed) since Tue 2026-06-09 19:55:03 UTC; 2min 39s ago
    Trigger: n/a
   Triggers: ● watchlog.service

Jun 09 19:55:03 ubuntu01 systemd[1]: Started watchlog.timer - Run watchlog script every 30 second.

