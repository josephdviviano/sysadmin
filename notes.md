LFCE exam 2016
--------------

**startup scripts**

+ systemctl
+ /etc/init.d
+ update-rc.d
+ sysv-rc-conf

**kernel settings & modules**

+ sysctl
+ cat /proc/modules
+ lsmod
+ ls /lib/modules/*/kernel/drivers/net
+ modprobe ${modulename}
+ modprobe --remove ${modulename}

**udev**

+ /etc/udev/rules.d
+ SUBSYSTEM=="usb", SYMLINK+="myusb"

**encryption**

+ cryptsetup luksFormat partition
+ cryptsetup luksOpen partition secret-disk
+ /etc/crypttab: secret-disk partition
+ mkfs.ext4 partition
+ /etc/fstab: partition mountpoint ext4 defaults 0 0
+ NB: for swap, /etc/crypttab: swapcrypt /dev/sda11 /dev/urandom wipes swap at boot

**tempfs**

+ ramdisk
+ mount -t tmpfs none /mnt/tmpfs

**quotas**

+ edit /etc/fstab to add `usrquota` mount option
+ remount filesystem
+ `quotaon /path/to/partition` -- enable quota
+ `quotacheck -cum /path/to/partition` -- initiates (overwrites) a quota file
+ `edquota -u ${username}` -- edit quoats

**swap**

mkswap -- format swap partition
swapon/swapoff -- enable/disable swap partition

**filesystem tuning**

+ e4defrag -- linux defrag for ext4
+ dumpe2fs -- check settings
+ tune2fs -- set settings

**apparmor**

+ all profiles in /etc/apparmor.d`
+ `apparmor_status`
+ `aa-complain /path/to/bin`
+ `aa-enforce /path/to/bin`

**MBR**

+ backup: `dd if=/dev/sda of=mbr.bkp bs=512 count=1`

**XFS**

+ xfsdump
+ xfsrestore

**logical volumes**

+ workflow: make physical volume, then volume group, then logical volume.
+ vgcreate
+ vgextend
+ vgreduce
+ pvcreate
+ pvdisplay
+ pvmove
+ pvremove
+ resize2fs

+ lvcreate -l 128 -s -n mysnap /dev/vg/mylvm

**RAID**

+ mdadm --create /dev/md0 --level=1 --raid-disks=2 /dev/sdbX /dev/sbcX
+ cat /proc/mdstat
+ mdadm service
+ mdadm --create /dev/md0 -l 5 -n 3 -x 1 /dev/sda2 /dev/sda3 /dev/sda4 /dev/sda5
+ mdadm --detail --scan >> /etc/mdadm.conf

# test failures
+ mdadm --fail /dev/md0 /dev/sdb2
+ mdadm --remove /dev/md0 /dev/sdb2
+ mdadm --add /dev/md0 /dev/sde2

**security**

+ chmod g+s filename -- setuid of file so group has privledges of owner
+ chmod g+s directory -- all files made inside dir will have the g+s group, regardless of who made them

**selinux**

+ getenforce
+ setenforce
+ vi /etc/selinux/config
+ most programs: -z at command line
+ restorecon resets file contexts
+ semanage fcontext
+ semanage boolean -l # settings
+ getsebool -a # -p for persistance across reboots
+ audit2allow
+ audit2why

**processes**

+ ulimit -a # show help, also /etc/security/limits.conf
+ ulimit -H -n ${num} # set HARD limit for number of open files
+ ulimit -S -n ${num} # set SOFT limit for number of open files
+ nice
+ renice
+ ldd
+ ldconfig # /etc/ld.so.conf
+ ipcs -p # show processes memory allocation and parent PIDs
+ ps aux | grep -e ${child_pid} -e ${parent_pid} # figure out the names of the processes from ipcs
+ any process in `ipcs` with nattach=0 is likely leaking memory and a zombie process

**signals**

+ kill
+ pkill

**benchmarking**

+ `bonnie++`
+ `bon_*` to convert files

**system monitoring**

/proc/sys

├── abi: binary info
├── debug: debug params, exception reporting
├── dev: device paramaters
├── fs: filesystem parameters (quota, file handles, innodes)
├── kernel: kernel parameters
├── net: network parameters (ipv4/6, netfilter, etc.)
├── sunrpc: ?
└── vm: virtual memory parameters

+ strace
+ uptime
+ top
+ ps
+ pstree
+ mpstat
+ iostat
+ numastat
+ free
+ cat /proc/${pid}/cmdline

**GRUB recovery**

```
set prefix=(disk,partition)/${grub_location}
set root=(disk,partition)
insmod normal
normal
```

**IO monitoring and tuning**

+ iostat: if %util approach 100 the system is I/O bound
+ iotop
+ ionice
+ elevator{cfq|deadline|noop}
+ cat /sys/block/${device}/queue/rotational == 0 if SDD (wear leveling req.)

**memory scheduling**

+ /proc/sys/vm -- see documentation @ /usr/src
+ /proc/sys/vm/overcommit_memory
+ /proc/${pid}/oom_score -- 'badness' of memory usage, for OOM killer
+ free
+ vmstat
+ pmap

**package management**

+ low: rpm, dpkg
+ high: yum, zypper, apt-get
+ /var/lib/rpm
+ rpm2cpio -- extract, don't install.
+ rpm -v -- verify integrity of installed software
+ /var/lib/dpkg
+ apt-get source logrotate -- download source code
+ yum: /etc/yum.repos.d/
+ `apt-cache`: queries of the repo database
+ `apt-cache search -n bash`: all packages with bash in their name
+ `apt-cache search bash`: all packages with a reference to bash
+ `apt-cache show bash`: show all details for bash
+ `apt-cache depends bash`: show dependencies for bash
+ apt-get: install / remove

**user account management**

+ /etc/skel -- default info
+ nologin -- lock account
+ sudo usermod -L ${username}
+ chage -E 2014-09-11 ${username} (time in past will expire, or lock, account)
+ change password in /etc/shadow to something invalid (e.g., !!)
+ vipw
+ /bin/bash -r : restricted shells (home folder only)
+ ln /bin/bash /bin/rbash
+ remote root admin: pam_securetty.so & /etc/securetty
+ vigr
+ getfacl file|directory
+ setfacl -m u:user:rw /home/user/file
+ /etc/pam.d/
+ list of PAM modules: ls /lib/*/security
+ usage: auth/account/session/password required/requisite/sufficient/optional pam_unix.sh params*

+ LDAP: system-config-authentication, authconfig-tui
+ tar --create --after-date '2011-12-1' -vzf backup.tar /var/tmp

**firewall**

+ firewalld firewall-cmd
+ zones, sources, services, ports
+ iptables, ufw
+ system-config-firewall
+ firewall-config
+ gufw
+ yast
+ /etc/firewalld, /usr/lib/firewalld
+ sudo firewall-cmd --permanent --zone=trusted --add-source=192.168.1.0/24
+ sudo firewall-cmd --permanent --zone=public --add-service=https
+ sudo firewall-cmd --permanent --get-services

**troubleshooting**

+ ifconfig, ip
+ lsmod
+ ping
+ route -n
+ debsums options some_package
+ aide --check

**backup**

+ CPIO:  cd /usr; find include | cpio -c -o > backup.cpio)
         cpio -id < backup.cpio
+ TAR:   cd /usr; tar zcvf backup.tar.gz include
         tar xvf backup.tar.gz
+ RSYNC: rsync -av /usr/include

**apt**

+ apt-cache pkgnames "kernel"

**networking**

# aliases

```
auto eth0
allow-hotplug eth0
iface eth0 inet static
    address 192.168.1.42
    netmask 255.255.255.0
    gateway 192.168.1.1

auto eth0:0
allow-hotplug eth0:0
iface eth0:0 inet static
    address 192.168.1.43
    netmask 255.255.255.0

auto eth0:1
allow-hotplug eth0:1
iface eth0:1 inet static
    address 192.168.1.44
    netmask 255.255.255.0
```

**labs to revisit**

- [x] lab 3.1    -- boot into non-graphical mode
- [x] lab 4.1-2  -- startup services
- [x] lab 6.1    -- kernel parameters
- [x] lab 7.1    -- kernel modules
- [x] lab 8.1    -- udev
- [x] lab 10.1   -- encryption
- [x] lab 13.2   -- quotas
- [x] lab 14.2   -- filesystem tuning
- [x] lab 18.2   -- setuid on scripts
- [x] lab 19.1   -- selinux
+ [x] lab 20.1-2 -- memory monitoring
+ [x] lab 24.1   -- benchmarking
- [x] lab 32.2   -- apt-get
+ [x] lab 36.1   -- PAM
+ [x] lab 37.2   -- CPIO
+ [x] lab 39.3   -- network aliases
+ [x] lab 40.3   -- firewall

**links**

+ http://www.tecmint.com/sed-command-to-create-edit-and-manipulate-files-in-linux/

