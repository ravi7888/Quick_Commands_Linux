# Quick_Commands_Linux
Quick commands for Linux systems - CentOs-7.3


1. Extend XFS filesystem on top of LVM

  1.1 'lsblk' :
  lists information about all available or the specified block devices.  The lsblk command reads the sysfs filesystem to         gather information.


  [root@vbmmapr1 ~]# lsblk
  NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
  sda           8:0    0  172G  0 disk 
  |-sda1        8:1    0    1G  0 part /boot
  |-sda2        8:2    0   31G  0 part 
  | |-cl-root 253:0    0 27.8G  0 lvm  /
  | `-cl-swap 253:1    0  3.2G  0 lvm  [SWAP]
  `-sda3        8:3    0  140G  0 part 
  sdb           8:16   0  100G  0 disk 
  sr0          11:0    1  4.1G  0 rom  
   
  1.2 'fdisk <device Name>' 
   fdisk /dev/sda
    Welcome to fdisk (util-linux 2.23.2).

    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): n
    Partition type:
    p   primary (2 primary, 0 extended, 2 free)
    e   extended
    Select (default p): p
    Partition number (3,4, default 3): 
    First sector (67108864-381681663, default 67108864): 
    Using default value 67108864
    Last sector, +sectors or +size{K,M,G} (67108864-381681663, default 381681663): 
    Using default value 381681663
    Partition 3 of type Linux and of size 150 GiB is set

    Command (m for help): t
    Partition number (1-3, default 3): 
    Hex code (type L to list all codes): 8e
    Changed type of partition 'Linux' to 'Linux LVM'

    Command (m for help): p

    Disk /dev/sda: 195.4 GB, 195421011968 bytes, 381681664 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x000a2f75

       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     2099199     1048576   83  Linux
    /dev/sda2         2099200    67108863    32504832   8e  Linux LVM
    /dev/sda3        67108864   381681663   157286400   8e  Linux LVM

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.

    WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)
    Syncing disks.

  [root@vbmmapr3 ~]# 'partprobe':
    partprobe is a program that informs the operating system kernel of partition table changes, by requesting that the     operating system re-read the partition table.

   
   
  
  1.3 'pvdisplay':
  pvdisplay shows the attributes of PVs, like size, physical extent size, space used for the VG descriptor area, etc.
  
  [root@vbmmapr3 ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               cl
  PV Size               31.00 GiB / not usable 3.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              7935
  Free PE               1
  Allocated PE          7934
  PV UUID               0Ijuj2-Hfph-sxZl-fA1w-N5Fj-2Q9Q-NP0z42
   
   1.4 'pvcreate' :
   pvcreate  initializes  a PV so that it is recognized as belonging to LVM, and allows the PV to be used in a VG. A PV can be    a disk partition, whole disk, meta device, or loopback file.
    
    [root@vbmmapr3 ~]# pvcreate /dev/sda3
    Physical volume "/dev/sda3" successfully created.

  1.5 'vgextend':
  vgextend adds one or more PVs to a VG. This increases the space available for LVs in the VG.
  
  
    [root@vbmmapr3 ~]# vgextend cl /dev/sda3
    Volume group "cl" successfully extended
  
  1.6 'vgdisplay':
  vgdisplay shows the attributes of VGs, and the associated PVs and LVs.
  
    [root@vbmmapr3 ~]# vgdisplay | grep -i free
    Free  PE / Size       38400 / 150.00 GiB
  
  1.7 'lvdisplay':
  lvdisplay shows the attributes of LVs, like size, read/write status, snapshot information, etc.

    [root@vbmmapr3 ~]# lvdisplay
      --- Logical volume ---
      LV Path                /dev/cl/swap
      LV Name                swap
      VG Name                cl
      LV UUID                Kusd6b-gaEx-B94M-DGnP-0X1x-Eh5T-WCuasW
      LV Write Access        read/write
      LV Creation host, time localhost.localdomain, 2018-02-22 21:06:05 +0800
      LV Status              available
      # open                 2
      LV Size                3.20 GiB
      Current LE             819
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     8192
      Block device           253:1
   
     --- Logical volume ---
      LV Path                /dev/cl/root
      LV Name                root
      VG Name                cl
      LV UUID                eV8AnA-vlUw-GfCy-5cNc-wrXs-ZQG7-nTwcme
      LV Write Access        read/write
      LV Creation host, time localhost.localdomain, 2018-02-22 21:06:05 +0800
      LV Status              available
      # open                 1
      LV Size                27.79 GiB
      Current LE             7115
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     8192
      Block device           253:0
   

  1.8 'lvextend': 
  Add space to a logical volume

    [root@vbmmapr3 ~]# lvextend -l +38400 /dev/cl/root
      Size of logical volume cl/root changed from 27.79 GiB (7115 extents) to 177.79 GiB (45515 extents).
      Logical volume cl/root successfully resized.
  
  
  1.9 'xfs_growfs':
  xfs_growfs  expands  an  existing XFS filesystem. The mount-point argument is the pathname of the directory where the         filesystem is mounted. The filesystem must be mounted to be grown. The existing contents of the filesystem are undisturbed,   and the added space becomes available for additional file storage. xfs_info is equivalent to invoking xfs_growfs with the -n   option.  data section is grown to the largest size possible with the -d option
    
    [root@vbmmapr3 ~]# xfs_growfs -d /
    meta-data=/dev/mapper/cl-root    isize=512    agcount=4, agsize=1821440 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=0 spinodes=0
    data     =                       bsize=4096   blocks=7285760, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
    log      =internal               bsize=4096   blocks=3557, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    data blocks changed from 7285760 to 46607360
    
   1.10 'df'
   df displays the amount of disk space available on the file system containing each file name argument.  If no file name
   is given, the space available on all currently mounted file systems is shown.
    
    [root@vbmmapr3 ~]# df -h
    Filesystem           Size  Used Avail Use% Mounted on
    /dev/mapper/cl-root  178G  1.1G  177G   1% /
    devtmpfs             7.8G     0  7.8G   0% /dev
    tmpfs                7.8G     0  7.8G   0% /dev/shm
    tmpfs                7.8G  8.4M  7.8G   1% /run
    tmpfs                7.8G     0  7.8G   0% /sys/fs/cgroup
    /dev/sda1           1014M  139M  876M  14% /boot
    tmpfs                1.6G     0  1.6G   0% /run/user/0
    [root@vbmmapr3 ~]# 
