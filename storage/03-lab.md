# **Lab: Logical Volume Management (LVM) Configuration on Linux**

## **Step 1 — Identify Available Disks**

First, verify which disks are available and unused.

```bash
lsblk
```

**Example output:**

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0    1G  0 part /boot
└─sda2   8:2    0   19G  0 part /
sdb      8:16   0    5G  0 disk
sdc      8:32   0    5G  0 disk
sdd      8:48   0    5G  0 disk
sde      8:64   0    5G  0 disk
```

You can also use:

```bash
sudo fdisk -l
```

 **Note:** Make sure `/dev/sdb` through `/dev/sde` do **not** contain existing partitions or filesystems.

 

## **Step 2 — Create Physical Volumes (PV)**

Initialize the 4 new disks for LVM.

```bash
sudo pvcreate /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

**Expected output:**

```
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.
```

Verify:

```bash
sudo pvs
```

Example:

```
  PV         VG   Fmt  Attr PSize  PFree
  /dev/sdb        lvm2    5.00g 5.00g
  /dev/sdc        lvm2    5.00g 5.00g
  /dev/sdd        lvm2    5.00g 5.00g
  /dev/sde        lvm2    5.00g 5.00g
```

 

## **Step 3 — Create a Volume Group (VG)**

Create a single volume group named `vg_storage` using all four physical volumes.

```bash
sudo vgcreate vg_storage /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

Verify:

```bash
sudo vgs
```

Example:

```
  VG         #PV #LV #SN Attr   VSize  VFree
  vg_storage   4   0   0 wz--n- 19.99g 19.99g
```

 

## **Step 4 — Create Logical Volumes (LV)**

We’ll create three LVs:

| Logical Volume | Size | Mount Point |
| -------------- ||    -- |
| lv_app1        | 10G             |  /app1       |
| lv_app2         | 15G             | /app2        |
| lv_backup      | Remaining space | /backup     |

>   If your VG is smaller, adjust sizes accordingly.

### 4.1 Create `lv_app1`

```bash
sudo lvcreate -L 10G -n lv_app1 vg_storage
```

### 4.2 Create `lv_app2`

```bash
sudo lvcreate -L 5G -n lv_app2 vg_storage
```

### 4.3 Create `lv_backup` (remaining space)

```bash
sudo lvcreate -l 100%FREE -n lv_backup vg_storage
```

Verify:

```bash
sudo lvs
```

Example:

```
  LV        VG         Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_backup vg_storage -wi-a -- 4.99g
  lv_app1   vg_storage -wi-a -- 10.00g
  lv_app2    vg_storage -wi-a -- 5.00g
```

 

## **Step 5 — Create Filesystems**

Format the new logical volumes with `ext4` (or `xfs` if preferred).

```bash
sudo mkfs.ext4 /dev/vg_storage/lv_app1
sudo mkfs.ext4 /dev/vg_storage/lv_app2
sudo mkfs.ext4 /dev/vg_storage/lv_backup
```

Check filesystem type and UUIDs:

```bash
sudo blkid /dev/vg_storage/*
```

 

## **Step 6 — Create Mount Points and Mount Filesystems**

```bash
sudo mkdir -p  /app1 /app2 /backup
sudo mount /dev/vg_storage/lv_app1  /app1
sudo mount /dev/vg_storage/lv_app2 /app2
sudo mount /dev/vg_storage/lv_backup /backup
```

Verify mounts:

```bash
df -h | grep vg_storage
```

Example:

```
/dev/mapper/vg_storage-lv_app1   10G  24M  9.9G  1%  /app1
/dev/mapper/vg_storage-lv_app2     5G   20M  5.0G  1% /app2
/dev/mapper/vg_storage-lv_backup  5G   20M  5.0G  1% /backup
```

 

## **Step 7 — Make Mounts Persistent in `/etc/fstab`**

Edit `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add these lines (use `blkid` UUIDs for reliability):

```
/dev/vg_storage/lv_app1    /app1    ext4   defaults  0 0
/dev/vg_storage/lv_app2    /app2     ext4   defaults  0 0
/dev/vg_storage/lv_backup /backup  ext4   defaults  0 0
```

Save and test:

```bash
sudo umount  /app1 /app2 /backup
sudo mount -a
```

If no error appears, verify:

```bash
df -h
```
 **Tip:** Always use `mount -a` after editing `/etc/fstab` to check for syntax errors before rebooting.

 

## **Step 8 — Verify the Configuration**

```bash
lsblk
df -h
sudo vgs
sudo lvs
sudo pvs
```

Example output snippet:

```
VG         #PV #LV #SN Attr   VSize  VFree
vg_storage   4   3   0 wz--n- 19.99g 0
```

 

## **Step 9 — Expansion Exercise (Add a New Disk)**

### 9.1 Add a new disk `/dev/sdf`

Identify the new disk:

```bash
lsblk
```

### 9.2 Create a Physical Volume

```bash
sudo pvcreate /dev/sdf
```

### 9.3 Extend the Volume Group

```bash
sudo vgextend vg_storage /dev/sdf
```

### 9.4 Extend a Logical Volume (`lv_backup` for example)

Extend by 5 GB:

```bash
sudo lvextend -L +5G /dev/vg_storage/lv_backup
```

Or use all free space:

```bash
sudo lvextend -l +100%FREE /dev/vg_storage/lv_backup
```

### 9.5 Resize Filesystem

```bash
sudo resize2fs /dev/vg_storage/lv_backup
```

Check:

```bash
df -h /backup
```

Output:

```
Filesystem                         Size  Used Avail Use%
/dev/mapper/vg_storage-lv_backup    10G   28M   10G   1%
```

 
