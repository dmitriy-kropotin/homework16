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
```
