# Step by step create LVM

#### Using root user 
`$ sudo su; 123456a@`

#### 1. Create physical volumes
`$ pvcreate /dev/sdb /dev/sdb`

#### 2. Display list physical volumes after created
`$ pvs -a`

```
root@ipos:/home/ipos# pvs -a
  PV                       VG        Fmt  Attr PSize   PFree
  /dev/loop0                              ---       0       0
  /dev/sda2                               ---       0       0
  /dev/sda3                ubuntu-vg lvm2 a--  <29.00g      0
  /dev/sdb                           lvm2 ---  500.00g 500.00g
  /dev/ubuntu-vg/lv-0                     ---       0       0
  /dev/ubuntu-vg/ubuntu-lv
```


#### 3. Create volume group name 'vg_minio_storage'
`$ vgcreate vg_minio_storage /dev/sdb`

#### 4. Display list volume groups after created
`$ vgs -a`

```
root@ipos:/home/ipos# vgs -a
  VG               #PV #LV #SN Attr   VSize    VFree
  ubuntu-vg          1   2   0 wz--n-  <29.00g       0
  vg_minio_storage   1   0   0 wz--n- <500.00g <500.00g
```



#### 5. Create logical volume name 'vol_minio'
`$ lvcreate -n vol_minio -l 100%FREE vg_minio_storage`

#### 6. Display list logical volumes after create
`$ lvs -a`

```
root@ipos:/home/ipos# lvs -a
  LV        VG               Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0      ubuntu-vg        -wi-ao----    4.00g
  ubuntu-lv ubuntu-vg        -wi-ao----  <25.00g
  vol_minio vg_minio_storage -wi-a----- <500.00g
```

#### 7. Create file system ext4 mapping with logical volume 'vol_minio'
`$ mkfs.ext4 /dev/vg_minio_storage/vol_minio`

```
root@ipos:/home/ipos# mkfs.ext4 /dev/vg_minio_storage/vol_minio
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 131070976 4k blocks and 32768000 inodes
Filesystem UUID: 5c91bce5-4e28-4b4c-bda4-1a5adc4ce21f
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

#### 8. Check filesystem type after created
`$ lsblk -f`

```
root@ipos:/home/ipos# lsblk -f
NAME                         FSTYPE      LABEL UUID                                   MOUNTPOINT
loop0                        squashfs                                                 /snap/core/7270
sda
├─sda1
├─sda2                       ext4              0a636231-f96c-48e9-a351-0c024943da43   /boot
└─sda3                       LVM2_member       H0r5Wa-CSXS-RVEv-adfy-PwuO-LuCx-AXtqqm
  ├─ubuntu--vg-ubuntu--lv    ext4              f83668bb-6f9f-4811-8d13-499c01e4692f   /
  └─ubuntu--vg-lv--0         swap              a1b81cc2-d5da-4b24-8f60-3c733637d509   [SWAP]
sdb                          LVM2_member       dEXiWF-9Y6K-0YZI-NHOg-2zJQ-DL9y-lvnjKR
└─vg_minio_storage-vol_minio ext4              5c91bce5-4e28-4b4c-bda4-1a5adc4ce21f
sr0
```

#### 9. Mounting logical volumes 
`$ blkid /dev/vg_minio_storage/vol_minio`

```
root@ipos:/home/ipos# blkid /dev/vg_minio_storage/vol_minio
/dev/vg_minio_storage/vol_minio: UUID="5c91bce5-4e28-4b4c-bda4-1a5adc4ce21f" TYPE="ext4"
```


##### Create mount point
`$ mkdir /data`

##### Insert into /etc/fstab and check after inserted
`$ echo 'UUID=5c91bce5-4e28-4b4c-bda4-1a5adc4ce21f /data ext4 defaults 0 0' >> /etc/fstab; cat /etc/fstab`

```
root@ipos:/home/ipos# cat /etc/fstab
UUID=a1b81cc2-d5da-4b24-8f60-3c733637d509 none swap sw 0 0
UUID=f83668bb-6f9f-4811-8d13-499c01e4692f / ext4 defaults 0 0
UUID=0a636231-f96c-48e9-a351-0c024943da43 /boot ext4 defaults 0 0
UUID=5c91bce5-4e28-4b4c-bda4-1a5adc4ce21f ext4 defaults 0 0
```

##### Save the changes and check after saved
`$ mount -a; mount | grep /data`

#### 10. Check again
`$ lsblk -f`

```
root@ipos:/home/ipos# lsblk -f
NAME                         FSTYPE      LABEL UUID                                   MOUNTPOINT
loop0                        squashfs                                                 /snap/core/7270
sda
├─sda1
├─sda2                       ext4              0a636231-f96c-48e9-a351-0c024943da43   /boot
└─sda3                       LVM2_member       H0r5Wa-CSXS-RVEv-adfy-PwuO-LuCx-AXtqqm
  ├─ubuntu--vg-ubuntu--lv    ext4              f83668bb-6f9f-4811-8d13-499c01e4692f   /
  └─ubuntu--vg-lv--0         swap              a1b81cc2-d5da-4b24-8f60-3c733637d509   [SWAP]
sdb                          LVM2_member       dEXiWF-9Y6K-0YZI-NHOg-2zJQ-DL9y-lvnjKR
└─vg_minio_storage-vol_minio ext4              5c91bce5-4e28-4b4c-bda4-1a5adc4ce21f   /data
sr0
```

> If you want resizing logical volumes or extending volume groups you can see link source below for detail <br>
> Source: https://www.tecmint.com/manage-and-create-lvm-parition-using-vgcreate-lvcreate-and-lvextend/
