# WordPress Web Solution

In this project I will be building a 3 tier architecture to host a WordPress site.

Below is my network IP and machine names.

## IP information
| Server/Machine Name  	|   IP	          | 
|---	                  |             ---	|
|  Client Machine 	    |192.168.1.110  	|
| Web Server 	          |192.168.1.112 	  |
| MariaDB	              |192.168.1.113   	|

## WordPress - Web Server Disk Partition:

### Disk Partition using LVM - pvcreate, vgcreate and lvcreate

```bash
'fdisk -l | more' output
```

```bash
[root@osboxes /]# fdisk -l | more
Disk /dev/sda: 500 GiB, 536870912000 bytes, 1048576000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xad15da28

Device     Boot     Start        End   Sectors  Size Id Type
/dev/sda1  *         2048    2099199   2097152    1G 83 Linux
/dev/sda2         2099200  497027071 494927872  236G 83 Linux
/dev/sda3       497027072  515901439  18874368    9G 82 Linux swap / Solaris
/dev/sda4       515901440 1048575999 532674560  254G  5 Extended
/dev/sda5       515903488 1048575999 532672512  254G 83 Linux

Disk /dev/sdc: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdd: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdb: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

### To partition the disk

gdisk /dev/sdc

```bash
[root@osboxes /]# gdisk /dev/sdc
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: MBR only
  BSD: not present
  APM: not present
  GPT: not present

***************************************************************
Found invalid GPT and valid MBR; converting MBR to GPT format
in memory. THIS OPERATION IS POTENTIALLY DESTRUCTIVE! Exit by
typing 'q' if you don't want to convert your MBR partitions
to GPT format!
***************************************************************
```

accessing help or manual for disk partition 

```bash
?
```

output:

```bash
***************************************************************
Found invalid GPT and valid MBR; converting MBR to GPT format
in memory. THIS OPERATION IS POTENTIALLY DESTRUCTIVE! Exit by
typing 'q' if you don't want to convert your MBR partitions
to GPT format!
***************************************************************

Command (? for help): ?
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu
```

Identified the command information to partition the new disk

new partition:

n

```bash
Command (? for help): n

```

partition command - p

partition number, default is 1, i pressed enter to accept default 

```bash
Partition number (1-128, default 1):
```

First sector, i chose default so I pressed Enter

```bash
First sector (34-16777182, default = 2048) or {+-}size{KMGTP}: 

```

Last sector, I chose default again I pressed Enter:

```bash
Last sector (2048-16777182, default = 16777182) or {+-}size{KMGTP}: 

```

Confirm the file system, I pressed Enter to accept default:

```bash
Hex code or GUID (L to show codes, Enter = 8300): 8e00
Changed type of partition to 'Linux LVM'
```

Pressed p to confirm:

```bash
Command (? for help): p
Disk /dev/sdc: 16777216 sectors, 8.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 4A4C76B8-1D61-493A-8E72-F4DC90B62EA8
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 16777182
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        16777182   8.0 GiB     8E00  Linux LVM
```

To complete the process, Write table to disk, I pressed the following and accepted the prompt: 

```bash
W
y
```

```bash
**Output:**

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdc.
The operation has completed successfully.
```

To verify the partition changes, volume created:

```bash
gdisk -l /dev/sdc
```

```bash
output:
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.
Disk /dev/sdc: 16777216 sectors, 8.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 4A4C76B8-1D61-493A-8E72-F4DC90B62EA8
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 16777182
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        16777182   8.0 GiB     8E00  Linux LVM
```

I repeat the same process for the remaining disks

fdisk -l

Disk /dev/sda: 500 GiB - main virtual server hard disk

Disk /dev/sdc: 8 GiB, 8589934592  - new disk volume added

Disk /dev/sdd: 8 GiB, 8589934592 bytes - 2x new disk volume

Disk /dev/sdb: 8 GiB, 8589934592 bytes - 3x new disk volume

```bash
fdisk -l
```

```bash
**Output:**

Disk /dev/sda: 500 GiB, 536870912000 bytes, 1048576000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xad15da28

Device     Boot     Start        End   Sectors  Size Id Type
/dev/sda1  *         2048    2099199   2097152    1G 83 Linux
/dev/sda2         2099200  497027071 494927872  236G 83 Linux
/dev/sda3       497027072  515901439  18874368    9G 82 Linux swap / Solaris
/dev/sda4       515901440 1048575999 532674560  254G  5 Extended
/dev/sda5       515903488 1048575999 532672512  254G 83 Linux

Disk /dev/sdc: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 4A4C76B8-1D61-493A-8E72-F4DC90B62EA8

Device     Start      End  Sectors Size Type
/dev/sdc1   2048 16777182 16775135   8G Linux LVM

Disk /dev/sdd: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 6103CED9-A084-47FF-AF6A-A3FE5F9EDA2C

Device     Start      End  Sectors Size Type
/dev/sdd1   2048 16777182 16775135   8G Linux LVM

Disk /dev/sdb: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 92A886A2-E992-4B6D-9AE3-E6B55496177D

Device     Start      End  Sectors Size Type
/dev/sdb1   2048 16777182 16775135   8G Linux LVM
```

Checking drive partition using lsblk:

```bash
lsblk -a
```

```bash
**Output:**

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  500G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0  236G  0 part /
├─sda3   8:3    0    9G  0 part [SWAP]
├─sda4   8:4    0    1K  0 part 
└─sda5   8:5    0  254G  0 part /home
sdb      8:16   0    8G  0 disk 
└─sdb1   8:17   0    8G  0 part 
sdc      8:32   0    8G  0 disk 
└─sdc1   8:33   0    8G  0 part 
sdd      8:48   0    8G  0 disk 
└─sdd1   8:49   0    8G  0 part
```

Now I will use lvmdiskscan utility to check for available storage for LVM

```bash
lvmdiskscan
```

```bash
**Output:**

	/dev/sda1 [       1.00 GiB] 
  /dev/sda2 [     236.00 GiB] 
  /dev/sda3 [       9.00 GiB] 
  /dev/sda5 [    <254.00 GiB] 
  /dev/sdb1 [      <8.00 GiB] 
  /dev/sdc1 [      <8.00 GiB] 
  /dev/sdd1 [      <8.00 GiB] 
  0 disks
  7 partitions
  0 LVM physical volume whole disks
  0 LVM physical volumes
```

Now I used pvcreate utility to mark the disks as LVM physical volumes

```bash
pvcreate /dev/sdc1
pvcreate /dev/sdb1
pvcreate /dev/sdd1
```

```bash
**Output**

  Physical volume "/dev/sdc1" successfully created.
 
  Physical volume "/dev/sdb1" successfully created.

  Physical volume "/dev/sdd1" successfully created**.**
```

Verify all Physical Volumes

```bash
pvdisplay
```

```bash
**Output

"/dev/sdb1" is a new physical volume of "<8.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name               
  PV Size               <8.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               jXNxYL-IUNC-oIfc-LZys-KNFa-s2Qc-cON4e6
   
  "/dev/sdc1" is a new physical volume of "<8.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc1
  VG Name               
  PV Size               <8.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               KGF8bq-CJZp-u3yv-pyC5-QKpI-LVii-DrF5KK
   
  "/dev/sdd1" is a new physical volume of "<8.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdd1
  VG Name               
  PV Size               <8.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               4ZmzSg-SV0F-Ac4n-gZbZ-Ld5R-cSDT-2a4sB3**
```

Create Volume Group

I will now use vgcreate utility to add the PVs to a volume group. Name the VG webdata-vg

```bash
vgcreate webdata-vg /dev/sdc1 /dev/sdb1 /dev/sdd1
```

Verifying Volume Group:

```bash
vgdisplay
```

```bash
**Output:**

  --- Volume group ---
  VG Name               webdata-vg
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <23.99 GiB
  PE Size               4.00 MiB
  Total PE              6141
  Alloc PE / Size       0 / 0   
  Free  PE / Size       6141 / <23.99 GiB
  VG UUID               1CNHgC-Sxmd-wAVw-CFgd-k0NI-sn7h-vYFB1A
```

I will now use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

```bash
lvcreate -n apps-lv --size 12G webdata-vg
lvcreate -n logs-lv --size 11.98G webdata-vg
```

```bash
**Output:**
Logical volume "apps-lv" created.

Rounding up size to full physical extent 11.98 GiB
Logical volume "logs-lv" created.
```

I will now Confirm the entire setup and explore the below commands

```bash
sudo vgdisplay -v *#view complete setup - VG, PV, and LV*
sudo lvs *#view logical volume summary*
sudo lsblk
```

```bash
**Output - #vgdisplay -v

  --- Volume group ---
  VG Name               webdata-vg
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <23.99 GiB
  PE Size               4.00 MiB
  Total PE              6141
  Alloc PE / Size       6139 / 23.98 GiB
  Free  PE / Size       2 / 8.00 MiB
  VG UUID               1CNHgC-Sxmd-wAVw-CFgd-k0NI-sn7h-vYFB1A
   
  --- Logical volume ---
  LV Path                /dev/webdata-vg/apps-lv
  LV Name                apps-lv
  VG Name                webdata-vg
  LV UUID                UEzprG-LxCD-cLgi-PQzm-fmwB-17Me-1c38NK
  LV Write Access        read/write
  LV Creation host, time osboxes, 2020-11-10 18:21:08 -0500
  LV Status              available
  # open                 0
  LV Size                12.00 GiB
  Current LE             3072
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
   
  --- Logical volume ---
  LV Path                /dev/webdata-vg/logs-lv
  LV Name                logs-lv
  VG Name                webdata-vg
  LV UUID                26aEqs-U7qz-PD8s-Taxd-BU7S-ldSA-Sw0qMc
  LV Write Access        read/write
  LV Creation host, time osboxes, 2020-11-10 18:26:35 -0500
  LV Status              available
  # open                 0
  LV Size                11.98 GiB
  Current LE             3067
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
   
  --- Physical volumes ---
  PV Name               /dev/sdc1     
  PV UUID               KGF8bq-CJZp-u3yv-pyC5-QKpI-LVii-DrF5KK
  PV Status             allocatable
  Total PE / Free PE    2047 / 0
   
  PV Name               /dev/sdd1     
  PV UUID               4ZmzSg-SV0F-Ac4n-gZbZ-Ld5R-cSDT-2a4sB3
  PV Status             allocatable
  Total PE / Free PE    2047 / 2
   
  PV Name               /dev/sdb1     
  PV UUID               jXNxYL-IUNC-oIfc-LZys-KNFa-s2Qc-cON4e6
  PV Status             allocatable
  Total PE / Free PE    2047 / 0**

```

```bash
**Output**: #sudo lvs

LV      VG         Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  apps-lv webdata-vg -wi-a----- 12.00g                                                    
  logs-lv webdata-vg -wi-a----- 11.98g
```

```bash
**Output: #sudo lsblk

[root@osboxes /]# sudo lsblk 
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0  500G  0 disk 
├─sda1                     8:1    0    1G  0 part /boot
├─sda2                     8:2    0  236G  0 part /
├─sda3                     8:3    0    9G  0 part [SWAP]
├─sda4                     8:4    0    1K  0 part 
└─sda5                     8:5    0  254G  0 part /home
sdb                        8:16   0    8G  0 disk 
└─sdb1                     8:17   0    8G  0 part 
  └─webdata--vg-logs--lv 253:1    0   12G  0 lvm  
sdc                        8:32   0    8G  0 disk 
└─sdc1                     8:33   0    8G  0 part 
  └─webdata--vg-apps--lv 253:0    0   12G  0 lvm  
sdd                        8:48   0    8G  0 disk 
└─sdd1                     8:49   0    8G  0 part 
  ├─webdata--vg-apps--lv 253:0    0   12G  0 lvm  
  └─webdata--vg-logs--lv 253:1    0   12G  0 lvm**
```

Assign FileSystem type

```bash
mkfs.ext4 /dev/webdata-vg/apps-lv
mkfs.ext4 /dev/webdata-vg/logs-lv
```

```bash
**Output:**
mke2fs 1.45.4 (23-Sep-2019)
Creating filesystem with 3145728 4k blocks and 786432 inodes
Filesystem UUID: 5278f631-aa4f-4a6e-910e-9fc5425a046f
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

----------------------------------------------------------------------------

mke2fs 1.45.4 (23-Sep-2019)
Creating filesystem with 3140608 4k blocks and 786432 inodes
Filesystem UUID: f89737d8-93c1-44fc-8ded-2c3dfa958983
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

Creating /var/www/html directory to store website files

```bash
mkdir -p /var/www/html
```

Create /home/recovery/logs to store backup of log data

```bash
mkdir -p /home/recovery/logs
```

Mount /var/www/html on apps-lv logical volume so that 

```bash
mount /dev/webdata-vg/apps-lv /var/www/html/
```

I will now use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

```bash
rsync -r /var/log /home/recovery/logs/
```

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why above step rsync is very crucial)

```bash
mount /dev/webdata-vg/logs-lv /var/log/
```

I will now update /etc/fstab file so that the mount configuration will persist upon restart of the server.

Enable mount during reboot

```bash
nano etc/fstab
```

I amended the file to include the mounts mentioned above.

```bash
#
# /etc/fstab
# Created by anaconda on Sat Jul  4 16:41:53 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=ae254fa5-acbb-4380-9898-47dc0d10fa98 /                       xfs     defaults        0 0
UUID=772a2e31-507b-4a21-ac34-facb10987cbd /boot                   ext4    defaults        1 2
UUID=ba422fe4-29b1-4f81-895c-813bcad8d2a3 /home                   xfs     defaults        0 0
UUID=d8c39e92-2f6a-4c60-b615-a75030151075 swap                    swap    defaults        0 0
/dev/webdata-vg/apps-lv /var/www/html/                            ext4    defaults        0 0
/dev/webdata-vg/logs-lv /var/log                                  ext4    defaults        0 0           
```

Saved and exit the file using CTRL+X and use Y to save the file.

reboot the server to check if mounted path persists

```bash
init 6 #restarts server
```

```bash
Output: #checking for mount info after reboot
[root@osboxes ~]# df -h
Filesystem                        Size  Used Avail Use% Mounted on
devtmpfs                          887M     0  887M   0% /dev
tmpfs                             914M     0  914M   0% /dev/shm
tmpfs                             914M  9.6M  904M   2% /run
tmpfs                             914M     0  914M   0% /sys/fs/cgroup
/dev/sda2                         236G  5.9G  231G   3% /
/dev/sda5                         254G  2.0G  252G   1% /home
/dev/mapper/webdata--vg-logs--lv   12G   67M   12G   1% /var/log
/dev/mapper/webdata--vg-apps--lv   12G   41M   12G   1% /var/www/html
/dev/sda1                         976M  263M  646M  29% /boot
tmpfs                             183M  1.2M  182M   1% /run/user/42
tmpfs                             183M  4.6M  179M   3% /run/user/1000 

```

## Database Server: WordPress - DB

I will repeat most of the steps from above except for amending the name for Volume Group and Logical Volume.

I have configured a second server in Virtual Box and it is called WordPress - DB

I have added 3 disks from virtual box to the Centos 8 server. 

Using terminal I use fdisk to check the added disk to get their mount location.

```bash
[root@osboxes /]# fdisk -l | more
Disk /dev/sdb: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdd: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sda: 500 GiB, 536870912000 bytes, 1048576000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xad15da28

Device     Boot     Start        End   Sectors  Size Id Type
/dev/sda1  *         2048    2099199   2097152    1G 83 Linux
/dev/sda2         2099200  497027071 494927872  236G 83 Linux
/dev/sda3       497027072  515901439  18874368    9G 82 Linux swap / Solaris
/dev/sda4       515901440 1048575999 532674560  254G  5 Extended
/dev/sda5       515903488 1048575999 532672512  254G 83 Linux

Disk /dev/sdc: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

I used `df -h` command to have a view and familiarise yourself with the disk configuration on the server

This is the output for df -h command:

```bash
[root@osboxes /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        887M     0  887M   0% /dev
tmpfs           914M     0  914M   0% /dev/shm
tmpfs           914M  9.6M  904M   2% /run
tmpfs           914M     0  914M   0% /sys/fs/cgroup
/dev/sda2       236G  6.0G  230G   3% /
/dev/sda5       254G  2.0G  252G   1% /home
/dev/sda1       976M  263M  646M  29% /boot
tmpfs           183M  1.2M  182M   1% /run/user/42
tmpfs           183M  4.6M  179M   3% /run/user/1000
/dev/sr0         59M   59M     0 100% /run/media/osboxes/VBox_GAs_6.1.17
```

I will now Use `gdisk` utility to create a single partition on each of the 3 disks(**1) /dev/sdb** **2) /dev/sdc 3) /dev/sdd)** also making sure the disk type is set to LVM using the hexcode for centos8 which is **8e00**

Using `lsblk` utility to view the newly configured partition on each of the 3 disks.

```bash
[root@osboxes /]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  500G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0  236G  0 part /
├─sda3   8:3    0    9G  0 part [SWAP]
├─sda4   8:4    0    1K  0 part 
└─sda5   8:5    0  254G  0 part /home
sdb      8:16   0    8G  0 disk 
└─sdb1   8:17   0    8G  0 part 
sdc      8:32   0    8G  0 disk 
└─sdc1   8:33   0    8G  0 part 
sdd      8:48   0    8G  0 disk 
└─sdd1   8:49   0    8G  0 part 
sr0     11:0    1 58.2M  0 rom  /run/media/osboxes/VBox_GAs_6.1.17
```

Using `lvmdiskscan` utility to check for available storage for LVM

```bash
[root@osboxes /]# lvmdiskscan
  /dev/sda1 [       1.00 GiB] 
  /dev/sda2 [     236.00 GiB] 
  /dev/sda3 [       9.00 GiB] 
  /dev/sda5 [    <254.00 GiB] 
  /dev/sdb1 [      <8.00 GiB] 
  /dev/sdc1 [      <8.00 GiB] 
  /dev/sdd1 [      <8.00 GiB] 
  0 disks
  7 partitions
  0 LVM physical volume whole disks
  0 LVM physical volumes
```

Using `pvcreate` utility to mark the disks as LVM physical volumes

```bash
[root@osboxes /]# pvcreate /dev/sdd1
  Physical volume "/dev/sdd1" successfully created.
[root@osboxes /]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
[root@osboxes /]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
```

Using `vgcreate` utility to add the PVs to a volume group. Name the VG **dbdata-vg**

```bash
vgcreate **dbdata-vg** /dev/sdc1 /dev/sdb1 /dev/sdd1

[root@osboxes /]# vgcreate dbdata-vg /dev/sdc1 /dev/sdb1 /dev/sdd1
  Physical volume "/dev/sdc1" successfully created.
  Volume group "dbdata-vg" successfully created
```

Using `lvcreate` utility to create 2 logical volumes. **db-lv** (***Use half of the PV size***), and **logs-lv** ***Use the remaining space of the PV size***. **NOTE**: db-lv will be used to store database files while, logs-lv will be used to store data for logs.

```bash
lvcreate -n db-lv --size 11.99G **dbdata-vg**
lvcreate -n logs-lv --size 11.98G **dbdata-vg

Output:

[root@osboxes /]# lvcreate -n db-lv --size 11.99G dbdata-vg
  Rounding up size to full physical extent 11.99 GiB
  Logical volume "db-lv" created.
[root@osboxes /]# lvcreate -n logs-lv --size 11.98G dbdata-vg
  Rounding up size to full physical extent 11.98 GiB
  Logical volume "logs-lv" created.
[root@osboxes /]#**
```

Verifying all steps and outputs, using the following commands below:

```bash
sudo vgdisplay -v *#view complete setup - VG, PV, and LV

**Output:
[root@osboxes /]# sudo vgdisplay -v
  --- Volume group ---
  VG Name               dbdata-vg
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <23.99 GiB
  PE Size               4.00 MiB
  Total PE              6141
  Alloc PE / Size       6137 / 23.97 GiB
  Free  PE / Size       4 / 16.00 MiB
  VG UUID               ytdQXp-yp2j-X2jN-WD5E-pPs2-a8BZ-kAdKay
   
  --- Logical volume ---
  LV Path                /dev/dbdata-vg/db-lv
  LV Name                db-lv
  VG Name                dbdata-vg
  LV UUID                OaKuVh-88sI-67cn-2uvA-3Ti7-TTQZ-yqsGab
  LV Write Access        read/write
  LV Creation host, time osboxes, 2020-11-10 20:13:39 -0500
  LV Status              available
  # open                 0
  LV Size                11.99 GiB
  Current LE             3070
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
   
  --- Logical volume ---
  LV Path                /dev/dbdata-vg/logs-lv
  LV Name                logs-lv
  VG Name                dbdata-vg
  LV UUID                cy5qDN-c1eq-zxEB-dChe-CCXw-U9yo-9S6l7a
  LV Write Access        read/write
  LV Creation host, time osboxes, 2020-11-10 20:13:52 -0500
  LV Status              available
  # open                 0
  LV Size                11.98 GiB
  Current LE             3067
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
   
  --- Physical volumes ---
  PV Name               /dev/sdb1     
  PV UUID               H1Iubw-Tc8q-xWTh-WbTR-59gO-lSnF-G42x95
  PV Status             allocatable
  Total PE / Free PE    2047 / 0
   
  PV Name               /dev/sdd1     
  PV UUID               f3yVmt-Sxwh-1o4T-8mkE-gF79-71Rp-1UqNF9
  PV Status             allocatable
  Total PE / Free PE    2047 / 4
   
  PV Name               /dev/sdc1     
  PV UUID               SyfjqS-3SxM-u3kR-teGg-15T4-59l5-cUirb0
  PV Status             allocatable
  Total PE / Free PE    2047 / 0***

sudo lvs *#view logical volume summary

**Output:
[root@osboxes /]# sudo lvs
  LV      VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  db-lv   dbdata-vg -wi-a----- 11.99g                                                    
  logs-lv dbdata-vg -wi-a----- 11.98g***

sudo lsblk

**Output: 
[root@osboxes /]# sudo lvs
  LV      VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  db-lv   dbdata-vg -wi-a----- 11.99g                                                    
  logs-lv dbdata-vg -wi-a----- 11.98g**
```

Using `mkfs.ext4` to format the logical volume with Ext4 filesystem

```bash
mkfs.ext4 /dev/**dbdata-vg**/db-lv
mkfs.ext4 /dev/**dbdata-vg**/logs-lv

**Output:**

mke2fs 1.45.4 (23-Sep-2019)
Creating filesystem with 3143680 4k blocks and 786432 inodes
Filesystem UUID: e51d3e09-ae88-4cac-bacf-877dbb718002
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@osboxes /]# mkfs.ext4 /dev/dbdata-vg/logs-lv
mke2fs 1.45.4 (23-Sep-2019)
Creating filesystem with 3140608 4k blocks and 786432 inodes
Filesystem UUID: 458a3b6d-de4b-4c8c-bb46-ff031e14724f
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

Creating **/db** directory to store database files

```bash
mkdir -p /db
```

Creating **/home/recovery/logs** to store backup of log data

```bash
mkdir -p /home/recovery/logs
```

Mount **/db** on **db-lv** logical volume

```bash
mount /dev/dbdata-vg/db-lv /db
```

Use `rsync` utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (*This is required before mounting the file system*)

```bash
rsync -r /var/log /home/recovery/logs/
```

Mount **/var/log** on **logs-lv** logical volume. (*Note that all the existing data on /var/log will be deleted. That is why step 15 above is very crucial*)

```bash
mount /dev/dbdata-vg/logs-lv /var/log/
```

Restore log files back into **/var/log** directory.

Update `/etc/fstab` file so that the mount configuration will persist upon restart of the server.

```bash
/dev/dbdata-vg/db-lv /db                            ext4    defaults        0 0
/dev/dbdata-vg/logs-lv /var/log                     ext4    defaults        0 0 
```

After restart of server:

```bash
Output:

[root@osboxes /]# df -h
Filesystem                       Size  Used Avail Use% Mounted on
devtmpfs                         887M     0  887M   0% /dev
tmpfs                            914M     0  914M   0% /dev/shm
tmpfs                            914M  9.6M  904M   2% /run
tmpfs                            914M     0  914M   0% /sys/fs/cgroup
/dev/sda2                        236G  5.9G  231G   3% /
/dev/sda5                        254G  2.0G  252G   1% /home
/dev/mapper/dbdata--vg-logs--lv   12G   41M   12G   1% /var/log
/dev/mapper/dbdata--vg-db--lv     12G   41M   12G   1% /db
/dev/sda1                        976M  263M  646M  29% /boot
tmpfs                            183M  1.2M  182M   1% /run/user/42
tmpfs                            183M  4.6M  179M   3% /run/user/1000
```

# **Step 3 — Install MySQL**

I logged in to my Database server.

To find out the ip, run the following command by replacing the network interface name:

```bash
ip addr show enp0s3 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

Installing Mariadb server and prerequisites

```bash
dnf install php-mysqlnd php-fpm mariadb-server tar curl php-json
```

Accessing mysql using root

```bash
mysql -u root -p
```

Create a new user on client machine/ip

```bash
CREATE USER 'dinul'@'192.168.1.112' IDENTIFIED BY 'November2020!';
```

Creating a new database 'wordpress' and giving 'dinul' access to the wordpress database

```bash
mysql> CREATE DATABASE wordpress;
mysql> GRANT ALL ON wordpress.* TO `dinul`@`192.168.1.112`;
mysql> FLUSH PRIVILEGES;
mysql> exit
```

To determine the location of the MySQL configuration file:

```bash
mysql --help | grep "Default options" -A 1
```

```bash
Output:
[root@osboxes ~]# mysql --help | grep "Default options" -A 1
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf
```

```bash
nano /etc/my.cnf
```

add the bind address of the db server ip.

```bash
bind-address=192.168.1.113
```

Restart the service

```bash
systemctl restart mysqld
```

Added Webserver IP and client ip to whitelist on firewall

```bash
firewall-cmd --permanent --add-source=192.168.1.112
firewall-cmd --permanent --add-source=192.168.1.110
```

Allow mysql port on firewall

```bash
firewall-cmd --permanent --add-service=mysql
```

To save settings:

```bash
firewall-cmd --reload
```

# **Step 4 — Install Wordpress**

To show server IP (replace enp0s3 by whichever your NIC is)

run the command:

```bash
ip addr show enp0s3 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

Installed prerequisites:

```bash
dnf install php-mysqlnd php-fpm httpd tar curl php-json
```

Allowing ports HTTP and HTTPS on the firewall:

```bash
firewall-cmd --permanent --zone=public --add-service=http 
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

Starting the Apache server

```bash
systemctl start httpd
```

Enable httpd on reboot

```bash
systemctl enable httpd
```

Downloaded and extracted WordPress. Start by downloading the WordPress installation package and extracting its content:

```bash
curl https://wordpress.org/latest.tar.gz --output wordpress.tar.gz
tar xf wordpress.tar.gz
```

Copied the extracted WordPress directory into the /var/www/html directory:

```bash
cp -r wordpress /var/www/html
```

I used the following command too switch off selinux

```bash
setenforce 0
```

Lastly in this step, changed permissions:

```bash
chown -R apache:apache /var/www/html/wordpress
```

Access WordPress installation wizard and perform the actual WordPress installation. Navigate your browser to [http://192.168.1.112/wordpress](http://localhost/wordpress)

I can now see the installation screen for wordpress, but I will leave this for now, as I have to install mariadb server and create mysql user and providing the right permission in order to get 3 tier architecture working.

I will open mysql port on firewall

```bash
firewall-cmd --permanent --add-service=mysql
```

I am now whitelisting my db server ip and my client ip

```bash
firewall-cmd --permanent --add-source=192.168.1.113
firewall-cmd --permanent --add-source=192.168.1.110
```

Saving the rule

```bash
firewall-cmd --reload
```

I will now create file wp-config.php by copying wp-config-sample.php

```bash
mv /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
```

I will now edit the content of the file wp-config.php

```bash
nano /var/www/html/wordpress/wp-config.php
```

Ensured the following information information is stored in the wp-config.php file

DB_Name

DB_User

DB_Password

DB_Host

```bash
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'dinul' );

/** MySQL database password */
define( 'DB_PASSWORD', 'November2020!' );

/** MySQL hostname */
define( 'DB_HOST', '192.168.1.113' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

Also 'Authentication unique keys and Salts' needed information copied from the url - [https://api.wordpress.org/secret-key/1.1/salt/](https://api.wordpress.org/secret-key/1.1/salt/)

Example of mine:

```bash
define('AUTH_KEY',         '|JJ6+A^zFBlUPe&bSVC]1p0#`v.mNc||><.KT&*YKlE[^SeD&r.}J0/Lmw(w<`[/');
define('SECURE_AUTH_KEY',  '*+e{V-+;tQYM%pr:9ul(T*+(-jQ(Q6n]lpX(bu^e|j;n2_Y9=df<q|WxrHzozzjA');
define('LOGGED_IN_KEY',    'UX^hpM|:QAKkP<$x$s|A,Km=A7NF3@z%+lH~Z^;~0|-^5rQ(]gi ,?)t){L.u>-D');
define('NONCE_KEY',        '!VEHsKK5e-bL=5O-UnnVn#nQXY-{j5=J/j &F!M$-mq[A,aT#AO=hYx7I,1n|/=_');
define('AUTH_SALT',        'A3:+>J2l{$OwN-sBtlL|O>D1<.mH1Y[VH?4lYIEs6)8Q*6bYDd6eLc)QR-xl;k)1');
define('SECURE_AUTH_SALT', 's1rIk:8+#3*Xzi~[y&$XytR|^0#^SW>P ]*>7AB:DC6<-EODzvm 6-^tO@*+Af`*');
define('LOGGED_IN_SALT',   '$6Lecdb9{SS|,OjjZajZ] *ZzyV1HywNFdV601yf|C|t JzW>d *~0)U1g}o7sA5');
define('NONCE_SALT',       '||v8Tp<AB,p0/nuGvJh-QkQO+~ejtIKEj&.;${h:@H/i[&,:UV.+XsalPs&gqa<a');
```

I now saved and exit the text file.

Restarted the httpd service.

# **Step 4 — Configure WordPress to connect to remote database.**

Logged in my Wordpress server.

I now test mysql user which I created.

```bash
mysql -u dinul -pNovember2020! -h 192.168.1.113 wordpress
```

This is the output:

```bash
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 23
Server version: 10.3.17-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [wordpress]>
```

Going back to my browser and visiting the wordpress site - http://192.168.1.112/wordpress

I can now see the set up section for WordPress.

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled.png)

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%201.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%201.png)

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%202.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%202.png)

Navigate back to [http://192.168.1.112/wordpress/](http://192.168.1.112/wordpress/)

I can now see the  test sample page.

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%203.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%203.png)

From my client machine(192.168.1.110) i get the following:

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%204.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%204.png)

---

---

---

---

---

---

---

## Problems encountered:

### 1) Error establishing a database connection

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%205.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%205.png)

I added the following code snippet to /var/www/html/wordpress/wp-config.php file

```bash
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
```

I restarted apache service

```bash
systemctl restart httpd
```

I went back to my browser and back to Wordpress setup page

I can now see exact error occurred to diagnose the problem.

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%206.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%206.png)

It turns out selinux needed to be switched off.

This was done by running command:

```bash
setenforce 0
```

### 2) Access denied for user 'dinul'@'192.168.1.112' to database 'wordpress'

This was the error identified, it turns out correct permissions was not set. This needs to be set on the database mysql server.

![WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%207.png](WordPress%20Web%20Solution%20ec8e9d39ad394d07b8efd5b04d941110/Untitled%207.png)

### 3) Unable to login as mysql user from the webserver.

 

Following from the previous error, another issue which was related to same issue is when logged onto Wordpress WebServer.

When i tried to login to the sql user created I encountered the error.

Following error appeared:

```bash
ERROR 1044 (42000): Access denied for user 'dinul'@'192.168.1.112' to database 'wordpress'
```

I logged on my Database server, logged in again as root mysql user and added the following permission to grant access.

```bash
mysql> GRANT ALL ON wordpress.* TO `dinul`@`192.168.1.112`;
mysql> FLUSH PRIVILEGES;
mysql> exit
```

This has resolved the issue.

Credit and References

[How To Install MySQL on CentOS 8 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-8)

[How to install Wordpress on RHEL 8 / CentOS 8 Linux - LinuxConfig.org](https://linuxconfig.org/install-wordpress-on-redhat-8)

[DevOps Career And Mentorship Experts - Darey.io](https://darey.io/)