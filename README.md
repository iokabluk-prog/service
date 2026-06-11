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
# Задача: Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта
# Устанавливаем spawn-fcgi и необходимые для него пакеты
ubuntuadmin@ubuntu01:~$ sudo apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y
# Создаем файл с настройками
ubuntuadmin@ubuntu01:~$ sudo mkdir -p /etc/spawn-fcgi/
ubuntuadmin@ubuntu01:~$ sudo nano /etc/spawn-fcgi/fcgi.conf
ubuntuadmin@ubuntu01:~$ sudo cat /etc/spawn-fcgi/fcgi.conf
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
ubuntuadmin@ubuntu01:~$ sudo nano /etc/systemd/system/spawn-fcgi.service
ubuntuadmin@ubuntu01:~$ cat > /etc/systemd/system/spawn-fcgi.service
-bash: /etc/systemd/system/spawn-fcgi.service: Permission denied
ubuntuadmin@ubuntu01:~$ sudo cat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
ubuntuadmin@ubuntu01:~$ systemctl start spawn-fcgi
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to start 'spawn-fcgi.service'.
Authenticating as: admin (ubuntuadmin)
Password:
==== AUTHENTICATION COMPLETE ====
ubuntuadmin@ubuntu01:~$ systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; preset: enabled)
     Active: active (running) since Thu 2026-06-11 19:42:28 UTC; 11s ago
   Main PID: 10195 (php-cgi)
      Tasks: 33 (limit: 4605)
     Memory: 14.8M (peak: 15.1M)
        CPU: 147ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─10195 /usr/bin/php-cgi
             ├─10196 /usr/bin/php-cgi
             ├─10197 /usr/bin/php-cgi
             ├─10198 /usr/bin/php-cgi
             ├─10199 /usr/bin/php-cgi
             ├─10200 /usr/bin/php-cgi
             ├─10201 /usr/bin/php-cgi
             ├─10202 /usr/bin/php-cgi
             ├─10203 /usr/bin/php-cgi
             ├─10204 /usr/bin/php-cgi
             ├─10205 /usr/bin/php-cgi
             ├─10206 /usr/bin/php-cgi
             ├─10207 /usr/bin/php-cgi
             ├─10208 /usr/bin/php-cgi
             ├─10209 /usr/bin/php-cgi
             ├─10210 /usr/bin/php-cgi
             ├─10211 /usr/bin/php-cgi
             ├─10212 /usr/bin/php-cgi
             ├─10213 /usr/bin/php-cgi
             ├─10214 /usr/bin/php-cgi
             ├─10215 /usr/bin/php-cgi
             ├─10216 /usr/bin/php-cgi
             ├─10217 /usr/bin/php-cgi
             ├─10218 /usr/bin/php-cgi
             ├─10219 /usr/bin/php-cgi
             ├─10220 /usr/bin/php-cgi
             ├─10221 /usr/bin/php-cgi
             ├─10222 /usr/bin/php-cgi
             ├─10223 /usr/bin/php-cgi
             ├─10224 /usr/bin/php-cgi
lines 1-38
# Задача: Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно
# Установим Nginx
ubuntuadmin@ubuntu01:~$ sudo apt install nginx -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  nginx-common
Suggested packages:
  fcgiwrap nginx-doc
The following NEW packages will be installed:
  nginx nginx-common
0 upgraded, 2 newly installed, 0 to remove and 166 not upgraded.
Need to get 568 kB of archives.
After this operation, 1,598 kB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 nginx-common all 1.24.0-2ubuntu7.11 [44.3 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 nginx amd64 1.24.0-2ubuntu7.11 [523 kB]
Fetched 568 kB in 0s (2,618 kB/s)
Preconfiguring packages ...
Selecting previously unselected package nginx-common.
(Reading database ... 88372 files and directories currently installed.)
Preparing to unpack .../nginx-common_1.24.0-2ubuntu7.11_all.deb ...
Unpacking nginx-common (1.24.0-2ubuntu7.11) ...
Selecting previously unselected package nginx.
Preparing to unpack .../nginx_1.24.0-2ubuntu7.11_amd64.deb ...
Unpacking nginx (1.24.0-2ubuntu7.11) ...
Setting up nginx-common (1.24.0-2ubuntu7.11) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
Could not execute systemctl:  at /usr/bin/deb-systemd-invoke line 148.
Setting up nginx (1.24.0-2ubuntu7.11) ...
Not attempting to start NGINX, port 80 is already in use.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for ufw (0.36.2-6) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
# Для запуска нескольких экземпляров сервиса модифицируем исходный service для использования различной конфигурации, а также PID-файлов. Для этого создадим новый Unit для работы с шаблонами (/etc/systemd/system/nginx@.service)
# Создаем  два файла конфигурации (/etc/nginx/nginx-first.conf, /etc/nginx/nginx-second.conf). Их можно сформировать из стандартного конфига /etc/nginx/nginx.conf
ubuntuadmin@ubuntu01:~$ sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf
ubuntuadmin@ubuntu01:~$ sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx-second.conf
ubuntuadmin@ubuntu01:~$ sudo cat /etc/nginx/nginx-first.conf
user www-data;
worker_processes auto;
pid /run/nginx-first.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    ## Basic Settings
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ## SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;  # Убраны устаревшие TLSv1 и TLSv1.1
    ssl_prefer_server_ciphers on;

    ## Logging
    access_log /var/log/nginx/access.log;

    ## Gzip
    gzip on;

    ## Virtual Host Configs (только одно подключение)
    #include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/sites-enabled/*;

    ## Example server block
    server {
        listen 9001;
        server_name _;
        root /var/www/html;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
ubuntuadmin@ubuntu01:~$ sudo systemctl restart nginx@first
ubuntuadmin@ubuntu01:~$ sudo systemctl status nginx@first
● nginx@first.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; preset: enabled)
     Active: active (running) since Thu 2026-06-11 21:08:08 UTC; 4s ago
       Docs: man:nginx(8)
    Process: 11659 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-first.conf -q -g daemon on; master_>
    Process: 11660 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process o>
   Main PID: 11662 (nginx)
      Tasks: 3 (limit: 4605)
     Memory: 2.3M (peak: 2.4M)
        CPU: 74ms
     CGroup: /system.slice/system-nginx.slice/nginx@first.service
             ├─11662 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; m>
             ├─11663 "nginx: worker process"
             └─11665 "nginx: worker process"
ubuntuadmin@ubuntu01:~$ sudo nginx -t -c /etc/nginx/nginx-second.conf
nginx: the configuration file /etc/nginx/nginx-second.conf syntax is ok
nginx: configuration file /etc/nginx/nginx-second.conf test is successful
ubuntuadmin@ubuntu01:~$ sudo cat /etc/nginx/nginx-second.conf
user www-data;
worker_processes auto;
pid /run/nginx-second.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    ## Basic Settings
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ## SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;  # Убраны устаревшие TLSv1 и TLSv1.1
    ssl_prefer_server_ciphers on;

    ## Logging
    access_log /var/log/nginx/access.log;

    ## Gzip
    gzip on;

    ## Virtual Host Configs (только одно подключение)
    #include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/sites-enabled/*;

    ## Example server block
    server {
        listen 9002;
        server_name _;
        root /var/www/html;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
ubuntuadmin@ubuntu01:~$ sudo systemctl restart nginx@second
ubuntuadmin@ubuntu01:~$ sudo systemctl status nginx@second
● nginx@second.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; preset: enabled)
     Active: active (running) since Thu 2026-06-11 21:14:57 UTC; 7s ago
       Docs: man:nginx(8)
    Process: 11795 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-second.conf -q -g daemon on; master>
    Process: 11798 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process >
   Main PID: 11799 (nginx)
      Tasks: 3 (limit: 4605)
     Memory: 2.3M (peak: 2.5M)
        CPU: 80ms
     CGroup: /system.slice/system-nginx.slice/nginx@second.service
             ├─11799 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; >
             ├─11800 "nginx: worker process"
             └─11801 "nginx: worker process"

