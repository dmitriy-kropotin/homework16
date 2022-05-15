# homework16

```
[root@homework16 ~]# cat /etc/sysconfig/watchlog
# Configuration file for my watchdog service
# Place it to /etc/sysconfig
# File and word in that file that we will be monit
WORD="sudo"
LOG=/var/log/secure
```
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
```
[root@homework16 ~]# ls -la /opt/watchsecurelog.sh
-rwxr-xr-x. 1 root root 191 May 12 13:39 /opt/watchsecurelog.sh
```

```
[root@homework16 ~]# cat /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchsecurelog.sh $WORD $LOG
```
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
```
[root@homework16 ~]# systemctl start watchlog.timer
[root@homework16 ~]# systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
   Loaded: loaded (/etc/systemd/system/watchlog.timer; disabled; vendor preset:>
   Active: active (elapsed) since Sun 2022-05-15 14:47:12 UTC; 2s ago
  Trigger: n/a

May 15 14:47:12 homework16 systemd[1]: Started Run watchlog script every 30 sec>

```
```
May 15 14:46:47 homework16 systemd[1]: Starting My watchlog service...
May 15 14:46:47 homework16 root[41803]: Sun May 15 14:46:47 UTC 2022 :run check sudoers sudo /var/log/secure
May 15 14:46:47 homework16 root[41806]: Sun May 15 14:46:47 UTC 2022: !!! sudoer found!!!
May 15 14:46:47 homework16 systemd[1]: watchlog.service: Succeeded.
```
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
