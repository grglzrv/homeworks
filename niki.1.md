Write down the commands needed to create a snapshot of a drive. Also explain the disk usage requirements for the the snapshot.
-------------------------------------------------------------------------------------------

Виртуална машина с 2 диска в RAID1 и lvm група zeus, в която няма свободно пространство.
За целта добавих още един виртуален диск, за да може да разширя групата и да се създаде безпроблемно снапшотът.

Текущо състояние:

	root@tintirimintiri:~# vgdisplay 
	  --- Volume group ---
	  VG Name               zeus
	  System ID             
	  Format                lvm2
	  Metadata Areas        2
	  Metadata Sequence No  19
	  VG Access             read/write
	  VG Status             resizable
	  MAX LV                0
	  Cur LV                2
	  Open LV               2
	  Max PV                0
	  Cur PV                2
	  Act PV                2
	  VG Size               15.99 GiB
	  PE Size               4.00 MiB
	  Total PE              4093
	  Alloc PE / Size       2046 / 7.99 GiB
	  Free  PE / Size       0 / 0
	  VG UUID               IysbdG-aojm-k0JB-WFZ0-mEbS-WZJf-M4k3bm


Новият виртуален диск съответно е sdc, с помощта на fdisk му създавам един primary дял върху целия диск.

Разширяване на текущия lvm:

	root@tintirimintiri:~# vgextend zeus /dev/sdc1
	  Volume group "zeus" successfully extended

и съответно резултата:

	root@tintirimintiri:~# vgdisplay 
	  --- Volume group ---
	  VG Name               zeus
	  System ID             
	  Format                lvm2
	  Metadata Areas        2
	  Metadata Sequence No  19
	  VG Access             read/write
	  VG Status             resizable
	  MAX LV                0
	  Cur LV                2
	  Open LV               2
	  Max PV                0
	  Cur PV                2
	  Act PV                2
	  VG Size               15.99 GiB
	  PE Size               4.00 MiB
	  Total PE              4093
	  Alloc PE / Size       2046 / 7.99 GiB
	  Free  PE / Size       2047 / 8.00 GiB
	  VG UUID               IysbdG-aojm-k0JB-WFZ0-mEbS-WZJf-M4k3bm


Създаване на снапшот ('s' параметъра е за snapshot, 'n' параметъра е за името на снапшота, 'L' размера на дяла)
	root@tintirimintiri:~# lvcreate -s -n snimka0 -L 512M /dev/zeus/boot
	  Logical volume "snimka0" created


Преглед:
	root@tintirimintiri:~# lvdisplay 
	  --- Logical volume ---
	  LV Path                /dev/zeus/root
	  LV Name                root
	  VG Name                zeus
	  LV UUID                eE8Y69-7yA7-fwz6-Itb1-Wcli-JCOU-3pFUFy
	  LV Write Access        read/write
	  LV Creation host, time tintirimintiri, 2013-07-01 08:41:59 -0400
	  LV snapshot status     source of
	                         snimka0 [active]
	  LV Status              available
	  # open                 1
	  LV Size                7.53 GiB
	  Current LE             1928
	  Segments               1
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     256
	  Block device           253:0
	   
	  --- Logical volume ---
	  LV Path                /dev/zeus/boot
	  LV Name                boot
	  VG Name                zeus
	  LV UUID                tRBgYH-3ZZD-EtqT-DZMD-98ma-K3A1-KqTO7n
	  LV Write Access        read/write
	  LV Creation host, time tintirimintiri, 2013-07-02 05:01:15 -0400
	  LV Status              available
	  # open                 1
	  LV Size                472.00 MiB
	  Current LE             118
	  Segments               2
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     256
	  Block device           253:1
	   
	  --- Logical volume ---
	  LV Path                /dev/zeus/snimka0
	  LV Name                snimka0
	  VG Name                zeus
	  LV UUID                FiOfVN-0KoY-KpnH-U8y1-SjiC-sZpN-0uZ3dC
	  LV Write Access        read/write
	  LV Creation host, time tintirimintiri, 2013-07-03 09:37:09 -0400
	  LV snapshot status     active destination for boot
	  LV Status              available
	  # open                 0
	  LV Size                472.00 MiB
	  Current LE             118
	  COW-table size         512.00 MiB
	  COW-table LE           128
	  Allocated to snapshot  0.00%
	  Snapshot chunk size    4.00 KiB
	  Segments               1
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     256
	  Block device           253:2

Преглед на текущо дисково пространство:
	root@tintirimintiri:~# df -h
	Filesystem             Size  Used Avail Use% Mounted on
	rootfs                 7.5G  1.4G  6.0G  19% /
	udev                    10M     0   10M   0% /dev
	tmpfs                  101M  312K  100M   1% /run
	/dev/mapper/zeus-root  7.5G  1.4G  6.0G  19% /
	tmpfs                  5.0M     0  5.0M   0% /run/lock
	tmpfs                  201M     0  201M   0% /run/shm
	/dev/mapper/zeus-boot  458M   29M  406M   7% /boot

Създаване на временен файл с големина ~200MB, та когато се възвърне от снапшот дяла да се види дали промените биват отразени:
	root@tintirimintiri:/boot# dd if=/dev/zero of=/boot/test bs=1M count=200
	200+0 records in
	200+0 records out
	209715200 bytes (210 MB) copied, 2.03707 s, 103 MB/s

Текущо състояние, заети 39.25% от снапшота:
	  --- Logical volume ---
	  LV Path                /dev/zeus/snimka0
	  LV Name                snimka0
	  VG Name                zeus
	  LV UUID                FiOfVN-0KoY-KpnH-U8y1-SjiC-sZpN-0uZ3dC
	  LV Write Access        read/write
	  LV Creation host, time tintirimintiri, 2013-07-03 09:37:09 -0400
	  LV snapshot status     active destination for boot
	  LV Status              available
	  # open                 0
	  LV Size                472.00 MiB
	  Current LE             118
	  COW-table size         512.00 MiB
	  COW-table LE           128
	  Allocated to snapshot  39.25%
	  Snapshot chunk size    4.00 KiB
	  Segments               1
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     256
	  Block device           253:2


Размонтиране:
	root@tintirimintiri:~# umount /dev/mapper/zeus-boot


Връщане на Boot от снапшот:
	root@tintirimintiri:~# lvconvert --merge /dev/mapper/zeus-snimka0
	  Merging of volume snimka0 started.
	  boot: Merged: 39.2%
	  boot: Merged: 0.0%
	  Merge of snapshot into logical volume boot has finished.
	  Logical volume "snimka0" successfully removed

Проверка:
	root@tintirimintiri:~# lvdisplay 
	  --- Logical volume ---
	  LV Path                /dev/zeus/root
	  LV Name                root
	  VG Name                zeus
	  LV UUID                eE8Y69-7yA7-fwz6-Itb1-Wcli-JCOU-3pFUFy
	  LV Write Access        read/write
	  LV Creation host, time tintirimintiri, 2013-07-01 08:41:59 -0400
	  LV Status              available
	  # open                 1
	  LV Size                7.53 GiB
	  Current LE             1928
	  Segments               1
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     256
	  Block device           253:0
	   
	  --- Logical volume ---
	  LV Path                /dev/zeus/boot
	  LV Name                boot
	  VG Name                zeus
	  LV UUID                tRBgYH-3ZZD-EtqT-DZMD-98ma-K3A1-KqTO7n
	  LV Write Access        read/write
	  LV Creation host, time tintirimintiri, 2013-07-02 05:01:15 -0400
	  LV Status              available
	  # open                 0
	  LV Size                472.00 MiB
	  Current LE             118
	  Segments               2
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     256
	  Block device           253:1


Монтиране на boot:
	root@tintirimintiri:~# mount -t ext4 /dev/zeus/boot /boot
но в случая е описан в fstab, затова мързеливата:
	root@tintirimintiri:~# mount -a

Създадения по горе временен файл от 210MB не същестува в boot дяла.

Важно е да се знае, че при връщане от снапшот, то той бива изтриван.

Ако снапшотът е монтиран, например в mnt, то може да се види първоначалният изглед на дяла.

Премахване на снапшот:
	root@tintirimintiri:~# lvremove /dev/zeus/snimka0 
	Do you really want to remove active logical volume snimka0? [y/n]: y
	  Logical volume "snimka0" successfully removed

Относно колко трябва да бъде въпросният снапшот е много относително, защото снапшотът съдържа в себе си промените по дяла, в моя случай на boot, където създадох един файл с големина 200MB.
Ако направя снапшот на root, съответно той трябва да е по-голям по размер, защото ако пусна след това например update на операционната система, то снапшотът може да се запълни и да стане
неизползваем.
Размера на снапшота трябва да се съобрази с използването на файловата система и количестово промени за времето, за което ще има снапшот.


