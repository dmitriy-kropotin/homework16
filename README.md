# homework16


[root@homework16 sysconfig]# cat watchlog
# Configuration file for my watchdog service
# Place it to /etc/sysconfig
# File and word in that file that we will be monit
WORD="session opened for user root"
LOG=/var/log/secure
