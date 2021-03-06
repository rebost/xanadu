============================
Some useful FreeBSD commands
============================

`sockstat <https://www.freebsd.org/cgi/man.cgi?iostat>`_
========

list open IPv46 sockets: ::

    sockstat -46



`systat <https://www.freebsd.org/cgi/man.cgi?query=systat&apropos=0&sektion=0&manpath=FreeBSD+10.2-RELEASE&arch=default&format=html>`_
======

**display system statistics**

Network connections: ::

    systat -netstat 1

Network traffic going through active interfaces: ::

    systat -ifstat 1

Processor use and disk throughput: ::

    systat -iostat 1

Tx/Rx for one of icmp, icmp6, ip, ip6, tcp (in example): ::

    systat -tcp 1



`gstat <https://www.freebsd.org/cgi/man.cgi?gstat>`_
======

print statistics about GEOM disks: ::

    gstat



`iostat <https://www.freebsd.org/cgi/man.cgi?iostat>`_
======

Display up  to 8 devices with the most I/O every second ad infinitum: ::

    iostat -h -n 8 -w 1



`lsof <https://www.freebsd.org/cgi/man.cgi?query=lsof&manpath=FreeBSD+10.2-RELEASE+and+Ports&format=html>`_
====

To list all open    Internet, x.25 (HP-UX), and UNIX domain files, use: ::

    lsof -i -U

To find the process that has /var/run/log open, use: ::

    lsof /var/run/log

To send a SIGHUP to the processes that have /var/run/log open, use: ::

    kill -HUP `lsof -t /u/abe/bar`



`pf <https://www.freebsd.org/cgi/man.cgi?query=pfctl&sektion=8&apropos=0&manpath=FreeBSD+10.2-RELEASE>`_
====

show pf info: ::

    pfctl -s info

enable pf: ::

    pfctl -e

disable pf (don't!): ::

    pfctl -d

parse and check pf rules sanity: ::

    pfctl -nf /etc/pf.conf

load pf rules: ::

    pfctl -f /etc/pf.conf

load pf rules and flush everything: ::

    pfctl -F all -f /etc/pf.conf

load pf rules and flush nat and rules: ::

    pfctl -F nat -F rules -f /etc/pf.conf

show all: ::

    pfctl -sa

show nat rules: ::

    pfctl -sn

show firewall rules: ::

    pfctl -sr
    pfctl -vvsr  # for more verbose output including rule counters, ID numbers, and so on

live capture of pf activity: ::

    tcpdump -netttti pflog0



`ntpctl <https://calomel.org/ntpd.html>`_
====

print out each ntp peer including their next polling time as well as the offset, delay and jitter in milliseconds: ::

    ntpctl -sa



`dtrace <https://www.freebsd.org/cgi/man.cgi?query=dtrace&apropos=0&sektion=0&manpath=FreeBSD+10.2-RELEASE&arch=default&format=html>`_
======

load kernel modules: ::

    kldload dtrace
    kldload dtraceall

examples: ::

    dtrace -n 'syscall:::'
    dtrace -n 'syscall:::entry'
    dtrace -n ':::entry'



`truss <https://www.freebsd.org/cgi/man.cgi?query=truss&sektion=>`_
=====

Follow the system calls used in echoing "hello": ::

    truss /bin/echo hello

Do the same, but put the output into a file: ::

    truss -o /tmp/truss.out /bin/echo hello

Follow an already-running process: ::

    truss -p 34



`netstat <https://www.freebsd.org/cgi/man.cgi?query=netstat&sektion=1>`_
=======

Show the routes table

    netstat -rn



netif
=====

restart FreeBSD network service

    /etc/rc.d/netif restart



routing
=======

restart FreeBSD routing service

    /etc/rc.d/routing restart



`file flags <https://www.freebsd.org/doc/handbook/permissions.html>`_
==========

see file flags

    ls -lo /usr/jails/basejail/lib

modify file flags (make sure you are in kern level -1)

    chflags -R nosunlink /usr/jails/basejail
    chflags -Rf nouarch /usr/jails/basejail
    chflags -Rf noschg /usr/jails/basejail



`drill <https://www.freebsd.org/cgi/man.cgi?query=drill&sektion=1>`_
=======

Do a reverse lookup on address 123.231.123.231

    drill -x 123.231.123.231



ZFS backup
==========

on the backup machine, prepare the filesystem that will receive the backup. Generally you will want to create it unmounted

    zfs create -u -o canmount=noauto -o mountpoint=/backup/hetzner02 tank/BACKUP/hetzner02


send a backup of 'tank' from hetzner02 to a backup sub-filesystem to be stored on hetzner01 (full snapshot path without pool name / not mounted / force rollback)

    TIMESTAMP=`date "+%Y%m%d%H%M%S"`
    zfs snapshot -r tank@${TIMESTAMP}
    zfs send -R tank@${TIMESTAMP} | ssh -p 22 hetzner01.docbase.net zfs receive -dvuF tank/BACKUP/hetzner02


on the backup machine, make sure all backed-up filesystems are not mounted automatically on reboot

    for fs in `zfs list -rH -o name tank/BACKUP/hetzner02`; do zfs set canmount=noauto ${fs}; done


check the status of your backed-up pools

    zfs list -r -d 1 -o name,used,mounted,canmount,mountpoint tank/BACKUP
    zfs list -r -o name,used,mounted,canmount,mountpoint tank/BACKUP/hetzner02


RSYNC
=====

rsync using a non-default port

    rsync -aAHvz  -e "ssh -p 4001" rproxy01.docbase.net:/etc/ssl/example01 /etc/ssl



POUDRIERE
=========

create options for jail 103amd64 using ports collection 'default', custom set 'server' for list of ports found in /usr/local/etc/poudriere.d/pkg_lists/server_pkg_list

    poudriere options -j 103amd64 -p default -z server -f /usr/local/etc/poudriere.d/pkg_lists/server_pkg_list


build packages in jail 103amd64 using ports collection 'default', custom set 'server' for list of ports found in /usr/local/etc/poudriere.d/pkg_lists/server_pkg_list

    poudriere bulk -j 103amd64 -p default -z server -f /usr/local/etc/poudriere.d/pkg_lists/server_pkg_list



MISC
====

remove windows line endings from a file, in place

    # ^M is ctrl + V + M
    sed -i "" -e "s/^M//g" myfile.txt
