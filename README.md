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
