# Oracle Linux 8 — Storage Management Lab


 


## Lab Steps

> Note: every command shown uses `sudo` where root permission is required. Replace `/dev/sdb` with the spare disk on your VM if different. Always double-check device names with `lsblk` before running commands.

### 1) List current disk partitions

1. Open a terminal.
2. Run:

```bash
sudo lsblk -o NAME,MAJ:MIN,RM,SIZE,RO,TYPE,MOUNTPOINT
```

**Expected example output (truncated):**

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
├─sda1   8:1    0  1.0G  0 part /boot
└─sda2   8:2    0   39G  0 part /
sdb      8:16   0   10G  0 disk
```

**Verification:** Confirm a spare disk (e.g., `sdb`) exists and has no important partitions. If `sdb` already has partitions you must use a different spare device.

 

### 2) Create a partition with `fdisk`

**Goal:** create a single partition `/dev/sdb1` that uses the whole disk.

1. Start `fdisk`:

```bash
sudo fdisk /dev/sdb
```

2. Follow these interactive steps inside `fdisk` (commands are the single-letter entries you type):

* Press `n` (new partition)
* Select `p` (primary)
* Partition number: press `Enter` (default 1)
* First sector: press `Enter` (default)
* Last sector: press `Enter` (to use the whole disk)
* Press `w` (write table to disk and exit)

3. Example interactive transcript (abbreviated):

```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-20971519, default 20971519): 
Created a new partition 1 of type 'Linux' and of size 10 GiB.
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
```

### 3) Create an `ext4` file system

1. Make an `ext4` file system on the new partition:

```bash
sudo mkfs.ext4 /dev/sdb1
```

2. Example output:

```
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 2621440 4k blocks and 65536 inodes
Filesystem UUID: 123e4567-89ab-cdef-0123-456789abcdef
Superblock backups stored on blocks: ...
Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```

3. Create a mount point and mount it:

```bash
sudo mkdir -p /mnt/labdata
sudo mount /dev/sdb1 /mnt/labdata
```

4. Optional: add a human-readable label:

```bash
sudo e2label /dev/sdb1 labdata
```

**Verification:** Confirm mount and filesystem:

```bash
mount | grep /mnt/labdata
df -h /mnt/labdata
lsblk -f
```

Expected `lsblk -f` output excerpt:

```
NAME   FSTYPE LABEL   UUID                                 MOUNTPOINT
sdb1   ext4   labdata 123e4567-89ab-cdef-0123-456789abcdef /mnt/labdata
```

 

### 4) Increase swap space (create and enable additional swap)

**Goal:** create a swap partition or swapfile. This section shows both methods; prefer **swapfile** if you need easier cleanup.

#### Option A — Create a swapfile (recommended for lab)

1. Create a 2G zero-filled file:

```bash
sudo fallocate -l 2G /swapfile_lab
# If fallocate not available: sudo dd if=/dev/zero of=/swapfile_lab bs=1M count=2048
```

2. Set correct permissions:

```bash
sudo chmod 600 /swapfile_lab
```

3. Make it swap:

```bash
sudo mkswap /swapfile_lab
```

4. Enable it:

```bash
sudo swapon /swapfile_lab
```

5. Verify:

```bash
sudo swapon --show
free -h
```

**Expected example `swapon --show`:**

```
NAME             TYPE  SIZE  USED PRIO
/swapfile_lab    file  2G    0B   -2
```

6. To make it persistent across reboots, add to `/etc/fstab`:

```bash
echo '/swapfile_lab none swap sw 0 0' | sudo tee -a /etc/fstab
```

#### Option B — Use a swap partition (if you created a partition, e.g., `/dev/sdb2`)

1. Create swap on the partition:

```bash
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2
```

2. Add `/dev/sdb2` to `/etc/fstab` if persistent swap is desired:

```
/dev/sdb2 none swap sw 0 0
```

**Verification:** Run:

```bash
sudo swapon --show
free -h
```

Confirm the new swap appears and total swap increased.

 

### 5) Using and testing the new file system (optional)

1. Create a test file on the mounted partition:

```bash
sudo touch /mnt/labdata/TEST_FILE
sudo ls -l /mnt/labdata/TEST_FILE
```

2. Write some data to confirm file system operates:

```bash
echo "lab test" | sudo tee /mnt/labdata/hello.txt
sudo cat /mnt/labdata/hello.txt
```

**Verification:** `ls -la /mnt/labdata` should show `TEST_FILE` and `hello.txt`.

 

## Cleanup — Safely remove partitions and swap (non-destructive to root disk)

> Aim: return the VM to the pre-lab state. Use the reverse of the steps you performed. Only touch the lab devices/files (e.g., `/dev/sdb`, `/swapfile_lab`, `/dev/sdb1`).

### A) Remove swapfile

1. Turn off the swapfile:

```bash
sudo swapoff /swapfile_lab
```

2. Remove it from `/etc/fstab` if you added it:

```bash
sudo sed -i '\|/swapfile_lab|d' /etc/fstab
```

3. Delete the file:

```bash
sudo rm -f /swapfile_lab
```

4. Verify:

```bash
sudo swapon --show   # should show nothing for /swapfile_lab
free -h              # verify swap decreased
```

### B) Unmount and remove filesystem/partition

1. Unmount the mount point:

```bash
sudo umount /mnt/labdata
```

2. (Optional) Remove mount directory:

```bash
sudo rmdir /mnt/labdata
```

3. If you created a partition (`/dev/sdb1`) and want to delete it:

```bash
sudo fdisk /dev/sdb
# inside fdisk:
# press d  (delete partition)
# choose partition number (e.g., 1)
# press w  (write changes and exit)
```

4. Inform kernel of partition changes:

```bash
sudo partprobe /dev/sdb
```

5. Verify:

```bash
sudo lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
```

You should see `/dev/sdb` with no partitions again.

 
