# homework16
1. Создам сервис, который будет раз в 30 секунд проверять журнал `/var/log/secure` на наличие запуска `sudo`
2. Для это создам файл конфигурации

```
[root@homework16 ~]# cat /etc/sysconfig/watchlog
# Configuration file for my watchdog service
# Place it to /etc/sysconfig
# File and word in that file that we will be monit
WORD="sudo"
LOG=/var/log/secure
```

3. Создам скрипт поиска слова в журнале, и записи события, если слово найдено

```
[root@homework16 ~]# cat /opt/watchsecurelog.sh
#!/bin/bash
WORD=$1
LOG=$2
DATE=`date`
logger $DATE" :run check sudoers "$1" "$2
#grep $WORD $LOG
if grep $WORD $LOG &> /dev/null
	then
		logger "$DATE: !!! sudoer found!!!"
	else
exit 0
fi
```

4. Добавляю права на запуск скрипта

```
[root@homework16 ~]# ls -la /opt/watchsecurelog.sh
-rwxr-xr-x. 1 root root 191 May 12 13:39 /opt/watchsecurelog.sh
```

5. Добавляю службу для запуска скрипта

```
[root@homework16 ~]# cat /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchsecurelog.sh $WORD $LOG
```

6. Добавляю юнит для запуска службы по таймеру.

```
[root@homework16 ~]# cat /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service
[Install]
WantedBy=multi-user.target
```

7. Запусаю, проверяю статус.

```
[root@homework16 ~]# systemctl start watchlog.timer
[root@homework16 ~]# systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
   Loaded: loaded (/etc/systemd/system/watchlog.timer; disabled; vendor preset:>
   Active: active (elapsed) since Sun 2022-05-15 14:47:12 UTC; 2s ago
  Trigger: n/a

May 15 14:47:12 homework16 systemd[1]: Started Run watchlog script every 30 sec>
```

8. Проверяю системный журнал. Сообщения есть! Работает

```
cat /var/log/messages
....
May 15 14:46:47 homework16 systemd[1]: Starting My watchlog service...
May 15 14:46:47 homework16 root[41803]: Sun May 15 14:46:47 UTC 2022 :run check sudoers sudo /var/log/secure
May 15 14:46:47 homework16 root[41806]: Sun May 15 14:46:47 UTC 2022: !!! sudoer found!!!
May 15 14:46:47 homework16 systemd[1]: watchlog.service: Succeeded.
....
```

9. Устнавлию `spawn-fcgi` для следующей части ДЗ

```
[root@homework16 ~]# dnf instal epel-release
...
[root@homework16 ~]# dnf install spawn-fcgi php php-cli
...
Installed:
  apr-1.6.3-12.el8.x86_64
  apr-util-1.6.1-6.el8.1.x86_64
  apr-util-bdb-1.6.1-6.el8.1.x86_64
  apr-util-openssl-1.6.1-6.el8.1.x86_64
  httpd-2.4.37-43.module+el8.5.0+747+83fae388.3.x86_64
  httpd-filesystem-2.4.37-43.module+el8.5.0+747+83fae388.3.noarch
  httpd-tools-2.4.37-43.module+el8.5.0+747+83fae388.3.x86_64
  mailcap-2.1.48-3.el8.noarch
  mod_http2-1.15.7-3.module+el8.5.0+695+1fa8055e.x86_64
  nginx-filesystem-1:1.14.1-9.module+el8.4.0+542+81547229.noarch
  php-7.2.24-1.module+el8.4.0+413+c9202dda.x86_64
  php-cli-7.2.24-1.module+el8.4.0+413+c9202dda.x86_64
  php-common-7.2.24-1.module+el8.4.0+413+c9202dda.x86_64
  php-fpm-7.2.24-1.module+el8.4.0+413+c9202dda.x86_64
  rocky-logos-httpd-85.0-3.el8.noarch
  spawn-fcgi-1.6.3-17.el8.x86_64

Complete!
```

10. 

```
[root@homework16 ~]# cat /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"

```
```
[root@homework16 ~]# cat /etc/systemd/system/spawn-fcgi.service
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
```
```
[root@homework16 ~]# systemctl start spawn-fcgi.service
[root@homework16 ~]# systemctl status spawn-fcgi.service
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-05-15 15:37:51 UTC; 18s ago
 Main PID: 64369 (php-cgi)
    Tasks: 33 (limit: 11402)
   Memory: 18.8M
   CGroup: /system.slice/spawn-fcgi.service
           ├─64369 /usr/bin/php-cgi
           ├─64370 /usr/bin/php-cgi
           ├─64371 /usr/bin/php-cgi
           ├─64372 /usr/bin/php-cgi
           ├─64373 /usr/bin/php-cgi
           ├─64374 /usr/bin/php-cgi
           ├─64375 /usr/bin/php-cgi
           ├─64376 /usr/bin/php-cgi
           ├─64377 /usr/bin/php-cgi
           ├─64378 /usr/bin/php-cgi
           ├─64379 /usr/bin/php-cgi
           ├─64380 /usr/bin/php-cgi
           ├─64381 /usr/bin/php-cgi
           ├─64382 /usr/bin/php-cgi
           ├─64383 /usr/bin/php-cgi
           ├─64384 /usr/bin/php-cgi
           ├─64385 /usr/bin/php-cgi
           ├─64386 /usr/bin/php-cgi
           ├─64387 /usr/bin/php-cgi
           ├─64388 /usr/bin/php-cgi
           ├─64389 /usr/bin/php-cgi
           ├─64390 /usr/bin/php-cgi
           ├─64391 /usr/bin/php-cgi
           ├─64392 /usr/bin/php-cgi
           ├─64393 /usr/bin/php-cgi
           ├─64394 /usr/bin/php-cgi
           ├─64395 /usr/bin/php-cgi
           ├─64396 /usr/bin/php-cgi
           ├─64397 /usr/bin/php-cgi
           ├─64398 /usr/bin/php-cgi
           ├─64399 /usr/bin/php-cgi
           ├─64400 /usr/bin/php-cgi
           └─64401 /usr/bin/php-cgi

May 15 15:37:51 homework16 systemd[1]: Started Spawn-fcgi startup service by Otus.
lines 19-42/42 (END)
```
```
[root@homework16 system]# cat /usr/lib/systemd/system/httpd@.service
# This is a template for httpd instances.
# See httpd@.service(8) for more information.

[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd@.service(8)

[Service]
Type=notify
Environment=LANG=C
Environment=HTTPD_INSTANCE=%i
ExecStartPre=/bin/mkdir -m 710 -p /run/httpd/instance-%i
ExecStartPre=/bin/chown root.apache /run/httpd/instance-%i
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND -f conf/%i.conf
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful -f conf/%i.conf
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

```
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/second.conf
```

```
[root@homework16 system]# cat /etc/httpd/conf/first.conf | grep -e "Listen" -e "PidFile"
# least PidFile.
# Listen: Allows you to bind Apache to specific IP addresses and/or
# Change this to Listen on specific IP addresses as shown below to
#Listen 12.34.56.78:80
Listen 80
PidFile /var/run/httpd-first.pid
```

```
[root@homework16 system]# cat /etc/httpd/conf/second.conf | grep -e "Listen" -e "PidFile"
# least PidFile.
# Listen: Allows you to bind Apache to specific IP addresses and/or
# Change this to Listen on specific IP addresses as shown below to
#Listen 12.34.56.78:80
Listen 8080
PidFile /var/run/httpd-second.pid
```

```
systemctl start httpd@first.service
systemctl start httpd@seconf.service
```

```
[root@homework16 system]# systemctl status httpd@
httpd@first.service   httpd@second.service  httpd@seconf.service
[root@homework16 system]# systemctl status httpd@first.service
● httpd@first.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-05-15 18:01:04 UTC; 1h 10min ago
     Docs: man:httpd@.service(8)
  Process: 65821 ExecStartPre=/bin/chown root.apache /run/httpd/instance-first (code=exited, status=0/SUCCESS)
  Process: 65819 ExecStartPre=/bin/mkdir -m 710 -p /run/httpd/instance-first (code=exited, status=0/SUCCESS)
 Main PID: 65823 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 11402)
   Memory: 26.3M
   CGroup: /system.slice/system-httpd.slice/httpd@first.service
           ├─65823 /usr/sbin/httpd -DFOREGROUND -f conf/first.conf
           ├─65825 /usr/sbin/httpd -DFOREGROUND -f conf/first.conf
           ├─65826 /usr/sbin/httpd -DFOREGROUND -f conf/first.conf
           ├─65827 /usr/sbin/httpd -DFOREGROUND -f conf/first.conf
           └─65828 /usr/sbin/httpd -DFOREGROUND -f conf/first.conf

May 15 18:01:04 homework16 systemd[1]: Starting The Apache HTTP Server...
May 15 18:01:04 homework16 httpd[65823]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'S>
May 15 18:01:04 homework16 systemd[1]: Started The Apache HTTP Server.
May 15 18:01:04 homework16 httpd[65823]: Server configured, listening on: port 80
```

```
[root@homework16 system]# systemctl status httpd@second.service
● httpd@second.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-05-15 18:01:16 UTC; 1h 11min ago
     Docs: man:httpd.service(8)
 Main PID: 66053 (httpd)
   Status: "Running, listening on: port 8080"
    Tasks: 213 (limit: 11402)
   Memory: 24.2M
   CGroup: /system.slice/system-httpd.slice/httpd@second.service
           ├─66053 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─66054 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─66055 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─66056 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           └─66057 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND

May 15 18:01:16 homework16 systemd[1]: Starting The Apache HTTP Server...
May 15 18:01:16 homework16 httpd[66053]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'S>
May 15 18:01:16 homework16 systemd[1]: Started The Apache HTTP Server.
May 15 18:01:16 homework16 httpd[66053]: Server configured, listening on: port 8080
```

```
[root@homework16 system]# ss -tulpn | grep httpd
tcp   LISTEN 0      128          0.0.0.0:8080      0.0.0.0:*    users:(("httpd",pid=66057,fd=3),("httpd",pid=66056,fd=3),("httpd",pid=66055,fd=3),("httpd",pid=66053,fd=3))
tcp   LISTEN 0      128          0.0.0.0:80        0.0.0.0:*    users:(("httpd",pid=65828,fd=3),("httpd",pid=65827,fd=3),("httpd",pid=65826,fd=3),("httpd",pid=65823,fd=3))
```
