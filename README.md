Доп. задание*

Подготовить Vagrantfile, который сразу собирает систему с подключенным рейдом и смонтированными разделами. 
После перезагрузки стенда разделы должны автоматически примонтироваться.

1. В пример был взят vagrantfail из предыдущего задания, где собирался RAID 10 из 6 дисков

2. В файл были добавлены строчки которые описаны в первом домашнем задании.

sudo mdadm --create --verbose /dev/md0 -l 10 -n 6 /dev/sd{b,c,d,e,f,g}
sudo mkdir /etc/mdadm
sudo cd /etc/mdadm
sudo touch mdadm.conf
sudo chmod 777 mdadm.conf
sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >>/etc/mdadm/mdadm.conf
sudo parted -s /dev/md0 mklabel gpt
sudo parted /dev/md0 mkpart primary ext4 0% 15%
sudo parted /dev/md0 mkpart primary ext4 15% 30%
sudo parted /dev/md0 mkpart primary ext4 30% 45%
sudo parted /dev/md0 mkpart primary ext4 45% 60%
sudo parted /dev/md0 mkpart primary ext4 60% 75%
sudo parted /dev/md0 mkpart primary ext4 75% 100%
for i in $(seq 1 6); do sudo mkfs.ext4 /dev/md0p$i; done
sudo mkdir -p /raid/part{1,2,3,4,5,6}
for i in $(seq 1 6); do sudo mount /dev/md0p$i /raid/part$i; done
for i in $(seq 1 6); do echo `sudo blkid /dev/md0p$i | awk '{print $2}'` /u0$i ext4 defaults 0 0 >> /etc/fstab; done


Из инетресного, в интернете и у вас в методичке советуют применять команду mdadm --detail --scan --verbose, но автор статьи не рекомендует, т.к. команда пишет в конфигурационный файл названия разделов, а они в некоторых случаях могут измениться, тогда RAID-массив не соберётся. А mdadm --detail --scan записывает UUID разделов, которые не изменятся

собственно ссылка на статью

https://internet-lab.ru/create_mdadm_conf

Запуск vagrantfile


user@ubuntu-vm:~/DZ-OTUS/DZ-2.1$ vagrant up
Bringing machine 'otuslinux' up with 'virtualbox' provider...
==> otuslinux: Importing base box 'centos/7'...
==> otuslinux: Matching MAC address for NAT networking...
==> otuslinux: Checking if box 'centos/7' version '2004.01' is up to date...
==> otuslinux: Setting the name of the VM: DZ-21_otuslinux_1683759302724_59631
==> otuslinux: Clearing any previously set network interfaces...
==> otuslinux: Preparing network interfaces based on configuration...
    otuslinux: Adapter 1: nat
    otuslinux: Adapter 2: hostonly
==> otuslinux: Forwarding ports...
    otuslinux: 22 (guest) => 2222 (host) (adapter 1)
==> otuslinux: Running 'pre-boot' VM customizations...
==> otuslinux: Booting VM...
==> otuslinux: Waiting for machine to boot. This may take a few minutes...
    otuslinux: SSH address: 127.0.0.1:2222
    otuslinux: SSH username: vagrant
    otuslinux: SSH auth method: private key
    otuslinux:
    otuslinux: Vagrant insecure key detected. Vagrant will automatically replace
    otuslinux: this with a newly generated keypair for better security.
    otuslinux:
    otuslinux: Inserting generated public key within guest...
    otuslinux: Removing insecure key from the guest if it's present...
    otuslinux: Key inserted! Disconnecting and reconnecting using new SSH key...
==> otuslinux: Machine booted and ready!
==> otuslinux: Checking for guest additions in VM...
    otuslinux: No guest additions were detected on the base box for this VM! Guest
    otuslinux: additions are required for forwarded ports, shared folders, host only
    otuslinux: networking, and more. If SSH fails on this machine, please install
    otuslinux: the guest additions and repackage the box to continue.
    otuslinux:
    otuslinux: This is not an error message; everything may continue to work properly,
    otuslinux: in which case you may ignore this message.
==> otuslinux: Setting hostname...
==> otuslinux: Configuring and enabling network interfaces...
==> otuslinux: Running provisioner: shell...
    otuslinux: Running: inline script
    otuslinux: Loaded plugins: fastestmirror
    otuslinux: Determining fastest mirrors
    otuslinux:  * base: centos.intergenia.de
    otuslinux:  * extras: mirror.23m.com
    otuslinux:  * updates: ftp.plusline.net
    otuslinux: Resolving Dependencies
    otuslinux: --> Running transaction check
    otuslinux: ---> Package gdisk.x86_64 0:0.8.10-3.el7 will be installed
    otuslinux: ---> Package hdparm.x86_64 0:9.43-5.el7 will be installed
    otuslinux: ---> Package mdadm.x86_64 0:4.1-9.el7_9 will be installed
    otuslinux: --> Processing Dependency: libreport-filesystem for package: mdadm-4.1-9.el7_9.x86_64
    otuslinux: ---> Package smartmontools.x86_64 1:7.0-2.el7 will be installed
    otuslinux: --> Processing Dependency: mailx for package: 1:smartmontools-7.0-2.el7.x86_64
    otuslinux: --> Running transaction check
    otuslinux: ---> Package libreport-filesystem.x86_64 0:2.1.11-53.el7.centos will be installed
    otuslinux: ---> Package mailx.x86_64 0:12.5-19.el7 will be installed
    otuslinux: --> Finished Dependency Resolution
    otuslinux:
    otuslinux: Dependencies Resolved
    otuslinux:
    otuslinux: ================================================================================
    otuslinux:  Package                  Arch       Version                  Repository   Size
    otuslinux: ================================================================================
    otuslinux: Installing:
    otuslinux:  gdisk                    x86_64     0.8.10-3.el7             base        190 k
    otuslinux:  hdparm                   x86_64     9.43-5.el7               base         83 k
    otuslinux:  mdadm                    x86_64     4.1-9.el7_9              updates     439 k
    otuslinux:  smartmontools            x86_64     1:7.0-2.el7              base        546 k
    otuslinux: Installing for dependencies:
    otuslinux:  libreport-filesystem     x86_64     2.1.11-53.el7.centos     base         41 k
    otuslinux:  mailx                    x86_64     12.5-19.el7              base        245 k
    otuslinux:
    otuslinux: Transaction Summary
    otuslinux: ================================================================================
    otuslinux: Install  4 Packages (+2 Dependent packages)
    otuslinux:
    otuslinux: Total download size: 1.5 M
    otuslinux: Installed size: 4.3 M
    otuslinux: Downloading packages:
    otuslinux: Public key for hdparm-9.43-5.el7.x86_64.rpm is not installed
    otuslinux: warning: /var/cache/yum/x86_64/7/base/packages/hdparm-9.43-5.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
    otuslinux: Public key for mdadm-4.1-9.el7_9.x86_64.rpm is not installed
    otuslinux: http://ftp.hosteurope.de/mirror/centos.org/7.9.2009/os/x86_64/Packages/mailx-12.5-19.el7.x86_64.rpm: [Errno 12] Timeout on http://ftp.hosteurope.de/mirror/centos.org/7.9.2009/os/x86_64/Packages/mailx-12.5-19.el7.x86_64.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
    otuslinux: Trying other mirror.
    otuslinux: --------------------------------------------------------------------------------
    otuslinux: Total                                               50 kB/s | 1.5 MB  00:30
    otuslinux: Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    otuslinux: Importing GPG key 0xF4A80EB5:
    otuslinux:  Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
    otuslinux:  Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
    otuslinux:  Package    : centos-release-7-8.2003.0.el7.centos.x86_64 (@anaconda)
    otuslinux:  From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    otuslinux: Running transaction check
    otuslinux: Running transaction test
    otuslinux: Transaction test succeeded
    otuslinux: Running transaction
    otuslinux:   Installing : libreport-filesystem-2.1.11-53.el7.centos.x86_64             1/6
    otuslinux:   Installing : mailx-12.5-19.el7.x86_64                                     2/6
    otuslinux:   Installing : 1:smartmontools-7.0-2.el7.x86_64                             3/6
    otuslinux:   Installing : mdadm-4.1-9.el7_9.x86_64                                     4/6
    otuslinux:   Installing : hdparm-9.43-5.el7.x86_64                                     5/6
    otuslinux:   Installing : gdisk-0.8.10-3.el7.x86_64                                    6/6
    otuslinux:   Verifying  : mdadm-4.1-9.el7_9.x86_64                                     1/6
    otuslinux:   Verifying  : 1:smartmontools-7.0-2.el7.x86_64                             2/6
    otuslinux:   Verifying  : gdisk-0.8.10-3.el7.x86_64                                    3/6
    otuslinux:   Verifying  : mailx-12.5-19.el7.x86_64                                     4/6
    otuslinux:   Verifying  : hdparm-9.43-5.el7.x86_64                                     5/6
    otuslinux:   Verifying  : libreport-filesystem-2.1.11-53.el7.centos.x86_64             6/6
    otuslinux:
    otuslinux: Installed:
    otuslinux:   gdisk.x86_64 0:0.8.10-3.el7          hdparm.x86_64 0:9.43-5.el7
    otuslinux:   mdadm.x86_64 0:4.1-9.el7_9           smartmontools.x86_64 1:7.0-2.el7
    otuslinux:
    otuslinux: Dependency Installed:
    otuslinux:   libreport-filesystem.x86_64 0:2.1.11-53.el7.centos mailx.x86_64 0:12.5-19.el7
    otuslinux:
    otuslinux: Complete!
    otuslinux: mdadm: layout defaults to n2
    otuslinux: mdadm: layout defaults to n2
    otuslinux: mdadm: chunk size defaults to 512K
    otuslinux: mdadm: size set to 253952K
    otuslinux: mdadm: Defaulting to version 1.2 metadata
    otuslinux: mdadm: array /dev/md0 started.
    otuslinux: Information: You may need to update /etc/fstab.
    otuslinux:
    otuslinux: Information: You may need to update /etc/fstab.
    otuslinux:
    otuslinux: Information: You may need to update /etc/fstab.
    otuslinux:
    otuslinux: Information: You may need to update /etc/fstab.
    otuslinux:
    otuslinux: Information: You may need to update /etc/fstab.
    otuslinux:
    otuslinux: Information: You may need to update /etc/fstab.
    otuslinux:
    otuslinux: mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
    otuslinux: OS type: Linux
    otuslinux: Block size=1024 (log=0)
    otuslinux: Fragment size=1024 (log=0)
    otuslinux: Stride=512 blocks, Stripe width=1536 blocks
    otuslinux: 28112 inodes, 112128 blocks
    otuslinux: 5606 blocks (5.00%) reserved for the super user
    otuslinux: First data block=1
    otuslinux: Maximum filesystem blocks=33685504
    otuslinux: 14 block groups
    otuslinux: 8192 blocks per group, 8192 fragments per group
    otuslinux: 2008 inodes per group
    otuslinux: Superblock backups stored on blocks:
    otuslinux:  8193, 24577, 40961, 57345, 73729
    otuslinux:
    otuslinux: Allocating group tables: done
    otuslinux: Writing inode tables: done
    otuslinux: Creating journal (4096 blocks): done
    otuslinux: Writing superblocks and filesystem accounting information: done
    otuslinux:
    otuslinux: mke2fs 1.42.9 (28-Dec-2013)
    otuslinux: Filesystem label=
    otuslinux: OS type: Linux
    otuslinux: Block size=1024 (log=0)
    otuslinux: Fragment size=1024 (log=0)
    otuslinux: Stride=512 blocks, Stripe width=1536 blocks
    otuslinux: 28800 inodes, 115200 blocks
    otuslinux: 5760 blocks (5.00%) reserved for the super user
    otuslinux: First data block=1
    otuslinux: Maximum filesystem blocks=33685504
    otuslinux: 15 block groups
    otuslinux: 8192 blocks per group, 8192 fragments per group
    otuslinux: 1920 inodes per group
    otuslinux: Superblock backups stored on blocks:
    otuslinux:  8193, 24577, 40961, 57345, 73729
    otuslinux:
    otuslinux: Allocating group tables: done
    otuslinux: Writing inode tables: done
    otuslinux: Creating journal (4096 blocks): done
    otuslinux: Writing superblocks and filesystem accounting information: done
    otuslinux:
    otuslinux: mke2fs 1.42.9 (28-Dec-2013)
    otuslinux: Filesystem label=
    otuslinux: OS type: Linux
    otuslinux: Block size=1024 (log=0)
    otuslinux: Fragment size=1024 (log=0)
    otuslinux: Stride=512 blocks, Stripe width=1536 blocks
    otuslinux: 28448 inodes, 113664 blocks
    otuslinux: 5683 blocks (5.00%) reserved for the super user
    otuslinux: First data block=1
    otuslinux: Maximum filesystem blocks=33685504
    otuslinux: 14 block groups
    otuslinux: 8192 blocks per group, 8192 fragments per group
    otuslinux: 2032 inodes per group
    otuslinux: Superblock backups stored on blocks:
    otuslinux:  8193, 24577, 40961, 57345, 73729
    otuslinux:
    otuslinux: Allocating group tables: done
    otuslinux: Writing inode tables: done
    otuslinux: Creating journal (4096 blocks): done
    otuslinux: Writing superblocks and filesystem accounting information: done
    otuslinux:
    otuslinux: mke2fs 1.42.9 (28-Dec-2013)
    otuslinux: Filesystem label=
    otuslinux: OS type: Linux
    otuslinux: Block size=1024 (log=0)
    otuslinux: Fragment size=1024 (log=0)
    otuslinux: Stride=512 blocks, Stripe width=1536 blocks
    otuslinux: 28800 inodes, 115200 blocks
    otuslinux: 5760 blocks (5.00%) reserved for the super user
    otuslinux: First data block=1
    otuslinux: Maximum filesystem blocks=33685504
    otuslinux: 15 block groups
    otuslinux: 8192 blocks per group, 8192 fragments per group
    otuslinux: 1920 inodes per group
    otuslinux: Superblock backups stored on blocks:
    otuslinux:  8193, 24577, 40961, 57345, 73729
    otuslinux:
    otuslinux: Allocating group tables: done
    otuslinux: Writing inode tables: done
    otuslinux: Creating journal (4096 blocks): done
    otuslinux: Writing superblocks and filesystem accounting information: done
    otuslinux:
    otuslinux: mke2fs 1.42.9 (28-Dec-2013)
    otuslinux: Filesystem label=
    otuslinux: OS type: Linux
    otuslinux: Block size=1024 (log=0)
    otuslinux: Fragment size=1024 (log=0)
    otuslinux: Stride=512 blocks, Stripe width=1536 blocks
    otuslinux: 28448 inodes, 113664 blocks
    otuslinux: 5683 blocks (5.00%) reserved for the super user
    otuslinux: First data block=1
    otuslinux: Maximum filesystem blocks=33685504
    otuslinux: 14 block groups
    otuslinux: 8192 blocks per group, 8192 fragments per group
    otuslinux: 2032 inodes per group
    otuslinux: Superblock backups stored on blocks:
    otuslinux:  8193, 24577, 40961, 57345, 73729
    otuslinux:
    otuslinux: Allocating group tables: done
    otuslinux: Writing inode tables: done
    otuslinux: Creating journal (4096 blocks): done
    otuslinux: Writing superblocks and filesystem accounting information: done
    otuslinux:
    otuslinux: mke2fs 1.42.9 (28-Dec-2013)
    otuslinux: Filesystem label=
    otuslinux: OS type: Linux
    otuslinux: Block size=1024 (log=0)
    otuslinux: Fragment size=1024 (log=0)
    otuslinux: Stride=512 blocks, Stripe width=1536 blocks
    otuslinux: 47232 inodes, 188928 blocks
    otuslinux: 9446 blocks (5.00%) reserved for the super user
    otuslinux: First data block=1
    otuslinux: Maximum filesystem blocks=33816576
    otuslinux: 24 block groups
    otuslinux: 8192 blocks per group, 8192 fragments per group
    otuslinux: 1968 inodes per group
    otuslinux: Superblock backups stored on blocks:
    otuslinux:  8193, 24577, 40961, 57345, 73729
    otuslinux:
    otuslinux: Allocating group tables: done
    otuslinux: Writing inode tables: done
    otuslinux: Creating journal (4096 blocks): done
    otuslinux: Writing superblocks and filesystem accounting information: done
    otuslinux:
user@ubuntu-vm:~/DZ-OTUS/DZ-2.1$

