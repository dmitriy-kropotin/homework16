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
```
