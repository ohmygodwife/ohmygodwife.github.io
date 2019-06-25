---
layout: post
title: "linux command"
date: 2019-06-09 18:32:45 +0800
description: ""
category: command
tags: []
---

```shell
#show epoch time
date +%s

#sort file and list epoch time
ls -t | xargs stat -c%n:%Y

find jlib -name "*.jar" | awk '{print "jar -tvf "$1}' | sh -x | grep "ProtocolException.class" #find class from which jar
find -L dir -name "*.java" | xargs grep "ProtocolException" #search text in file

#decompress
tar zxvf nmap-i386-em-distro.tar.gz #x for decompress
unzip nmap-i386-em-distro.zip -d ./abc/
bzip2 -cd nmap.tar.bz2 | tar xvf -

#compress
tar zcvf nmap-i386-em-distro.tar.gz nmap #c for compress
zip -r nmap-i386-em-distro.zip nmap

#gdb with arguments
gdb --arg ./a.out [arg1, arg2]

#redirect stderr
{ make; } 2>1.txt

#check rpm version
rpm -qa pcre

#show ps header
ps aux |head -1; ps aux |grep nmxl
ps -eO ppid,user,ruser,euser,pcpu,pmem #-O indentical to -o pid,format,state,tname,time,command

#check disk
df -hl scan whole disk
df -h 查看每个根路径的分区大小
df -k
du -sh [目录名] 返回该目录的大小
du -sm [文件夹] 返回该文件夹总M数
du -am | sort -nr | head -n 10 #查找当前目录下占用空间最大的前10个文件

#read man, something like: syslog(3)
man 3 syslog

#check status of last command execution
echo $?

#view archive
objdump -a *.a
nm *.a
nm -IA a.out #list executable symbol name

#trace program system call
truss -f -o outfile [program]
strace -o outfile [program]

#Enable VNC from firewall
iptables -I INPUT -p tcp --dport 5801 -j ACCEPT

#Set env to make "sudo" command without pw
sudo visudo -f /etc/sudoers
	# User privilege specification 
	root   ALL=(ALL) ALL 
	test ALL=(ALL) NOPASSWD:ALL

#check service
chkconfig --list|grep telnet

#mount vituralbox sharefolder
mount -t vboxsf sharefolder /root/hello

#force rotate syslog file force
logrotate -vf /etc/logrotate.conf

#generate syslog record
logger -p local3.debug -f file #from file
logger -p local3.debug message

#check inode
ll -i /var/log/messages*

#memory leak check
valgrind --tool=memcheck --leak-check=full --show-reachable=yes --error-limit=no --log-file=proglog ./a.out
```

