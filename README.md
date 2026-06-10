# Задача: Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
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
ubuntuadmin@ubuntu01:~$ systemctl restart watchlog.timer
ubuntuadmin@ubuntu01:~$ systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
     Loaded: loaded (/etc/systemd/system/watchlog.timer; disabled; preset: enabled)
     Active: active (waiting) since Tue 2026-06-09 19:55:03 UTC; 7h ago
    Trigger: Wed 2026-06-10 03:14:33 UTC; 7s ago
   Triggers: ● watchlog.service

Jun 09 19:55:03 ubuntu01 systemd[1]: Started watchlog.timer - Run watchlog script every 30 second.
# Проверка
ubuntuadmin@ubuntu01:~$ tail -n 1000 /var/log/syslog  | grep word
2026-06-09T19:40:04.615403+00:00 ubuntu01 kernel: systemd[1]: Started systemd-ask-password-wall.path - Forward Password Requests to Wall Directory Watch.
2026-06-09T19:40:04.616014+00:00 ubuntu01 kernel: audit: type=1400 audit(1781033997.931:2): apparmor="STATUS" operation="profile_load" profile="unconfined" name="1password" pid=550 comm="apparmor_parser"
2026-06-10T03:12:50.812765+00:00 ubuntu01 root: Wed Jun 10 03:12:50 AM UTC 2026: I found word, Master!
2026-06-10T03:13:25.603533+00:00 ubuntu01 root: Wed Jun 10 03:13:25 AM UTC 2026: I found word, Master!
2026-06-10T03:14:03.525772+00:00 ubuntu01 root: Wed Jun 10 03:14:03 AM UTC 2026: I found word, Master!
2026-06-10T03:14:40.780298+00:00 ubuntu01 root: Wed Jun 10 03:14:40 AM UTC 2026: I found word, Master!
2026-06-10T03:15:25.622155+00:00 ubuntu01 root: Wed Jun 10 03:15:25 AM UTC 2026: I found word, Master!
2026-06-10T03:16:17.217119+00:00 ubuntu01 root: Wed Jun 10 03:16:17 AM UTC 2026: I found word, Master!
2026-06-10T03:16:55.609268+00:00 ubuntu01 root: Wed Jun 10 03:16:55 AM UTC 2026: I found word, Master!
2026-06-10T03:17:25.635043+00:00 ubuntu01 root: Wed Jun 10 03:17:25 AM UTC 2026: I found word, Master!
2026-06-10T03:18:05.545675+00:00 ubuntu01 root: Wed Jun 10 03:18:05 AM UTC 2026: I found word, Master!
2026-06-10T03:18:52.401648+00:00 ubuntu01 root: Wed Jun 10 03:18:52 AM UTC 2026: I found word, Master!
2026-06-10T03:19:25.604303+00:00 ubuntu01 root: Wed Jun 10 03:19:25 AM UTC 2026: I found word, Master!
2026-06-10T03:20:02.539385+00:00 ubuntu01 root: Wed Jun 10 03:20:02 AM UTC 2026: I found word, Master!
2026-06-10T03:20:44.495530+00:00 ubuntu01 root: Wed Jun 10 03:20:44 AM UTC 2026: I found word, Master!
