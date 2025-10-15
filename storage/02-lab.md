# LAB 02

## Initial Setup and Verification

### Step 1: Verify Available Storage Devices

1. Open a terminal and check the available block devices:
   ```bash
   lsblk
   ```
   You should see output similar to:
   ```
   NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda             8:0    0   46G  0 disk 
   ├─sda1          8:1    0  100M  0 part /boot/efi
   ├─sda2          8:2    0    1G  0 part /boot
   └─sda3          8:3    0 44.9G  0 part 
   sdb             8:16   0    5G  0 disk 
   sdc             8:32   0    5G  0 disk 
   ```

2. List the current partition tables on all disks:
   ```bash
   sudo fdisk -l | grep /dev/sd
   ```
   Your output will vary, but should show your main system disk (`sda`) and the two additional disks (`sdb`, `sdc`) without partitions.

### Step 2: Identify Mounted File Systems

1. Check currently mounted file systems:
   ```bash
   df -h
   ```

## Disk Partitioning

### Step 3: Create an MBR Partition on /dev/sdb

1. Start `fdisk` for `/dev/sdb`:
   ```bash
   sudo fdisk /dev/sdb
   ```

2. At the `Command` prompt, use these commands:
   - Type `n` to create a new partition
   - Type `p` for primary partition
   - Type `1` for partition number
   - Press `Enter` to accept the default first sector (2048)
   - Type `+2G` to create a 2GB partition
   - Type `p` to preview the partition table
   - Type `w` to write changes and exit 

3. Verify the partition was created:
   ```bash
   sudo fdisk -l /dev/sdb
   ```
   You should see output confirming `/dev/sdb1` exists.

### Step 4: Create a GPT Partition on /dev/sdc

1. Start `fdisk` for `/dev/sdc`:
   ```bash
   sudo fdisk /dev/sdc
   ```

2. At the `Command` prompt:
   - Type `g` to create a new GPT disk label
   - Type `n` to create a new partition
   - Type `1` for partition number
   - Press `Enter` twice to accept default first and last sectors (using full disk)
   - Type `p` to preview the partition table
   - Type `w` to write changes and exit 

3. Verify the partition was created:
   ```bash
   sudo fdisk -l /dev/sdc
   ```
   You should see output showing `/dev/sdc1` with a GPT label.

## File System Creation and Management

### Step 5: Create an ext4 File System on /dev/sdb1

1. Create the ext4 file system with a label:
   ```bash
   sudo mkfs -t ext4 -L Test /dev/sdb1
   ```

2. Display the block device attributes:
   ```bash
   sudo blkid /dev/sdb1
   ```
   Note the **UUID** and **LABEL** values in the output .

### Step 6: Create Mount Points and Mount File Systems

1. Create mount points:
   ```bash
   sudo mkdir /Test
   ```

2. Mount the file system temporarily:
   ```bash
   sudo mount /dev/sdb1 /Test
   ```

3. Verify the mount was successful:
   ```bash
   df -h /Test
   ```
   You should see output showing `/dev/sdb1` mounted on `/Test`.

### Step 7: Configure Persistent Mounting via /etc/fstab

1. Get the UUID of your partition:
   ```bash
   sudo blkid /dev/sdb1
   ```

2. Add an entry to `/etc/fstab` (replace the UUID with your actual UUID):
   ```bash
   echo "UUID=your-uuid-here /Test ext4 defaults 0 0" | sudo tee -a /etc/fstab
   ```

3. Test the fstab entry:
   ```bash
   sudo umount /Test
   sudo mount -a
   df -h /Test
   ```
   If successful, `/dev/sdb1` will be mounted on `/Test` .

## Swap Space Management

### Step 8: Increase Swap Space Using a Swap File

1. Check current swap space:
   ```bash
   swapon -s
   free -h
   ```

2. Create a 1GB swap file:
   ```bash
   sudo dd if=/dev/zero of=/.swapfile2 bs=1024 count=1048576
   ```

3. Set secure permissions on the swap file:
   ```bash
   sudo chmod 600 /.swapfile2
   ```

4. Initialize the swap file:
   ```bash
   sudo mkswap /.swapfile2
   ```

5. Enable the new swap file:
   ```bash
   sudo swapon /.swapfile2
   ```

6. Verify the additional swap space:
   ```bash
   swapon -s
   free -h
   ```
   You should see an increase in total available swap space .

### Step 9: Make the Swap File Permanent

1. Add the swap file to `/etc/fstab`:
   ```bash
   echo "/.swapfile2 swap swap defaults 0 0" | sudo tee -a /etc/fstab
   ```

2. Reload the system daemon:
   ```bash
   sudo systemctl daemon-reload
   ```

## Cleanup Operations

### Step 10: Remove Additional Swap Space

1. Disable the swap file:
   ```bash
   sudo swapoff /.swapfile2
   ```

2. Remove the swap file entry from `/etc/fstab`:
   ```bash
   sudo sed -i '/\.swapfile2/d' /etc/fstab
   ```

3. Remove the physical swap file:
   ```bash
   sudo rm -f /.swapfile2
   ```

4. Verify the swap file has been removed:
   ```bash
   swapon -s
   free -h
   ```
   The swap total should return to its original value .

### Step 11: Remove Partitions

1. Unmount the file system if mounted:
   ```bash
   sudo umount /Test
   ```

2. Remove the mount point:
   ```bash
   sudo rmdir /Test
   ```

3. Remove the fstab entry:
   ```bash
   sudo sed -i '/\/Test/d' /etc/fstab
   ```

4. Delete the partition on `/dev/sdb`:
   ```bash
   sudo fdisk /dev/sdb
   ```
   - Type `d` to delete the partition (automatically selects partition 1 if only one exists)
   - Type `w` to write changes and exit

5. Verify the partition has been removed:
   ```bash
   lsblk /dev/sdb
   ```
   The partition (`sdb1`) should no longer appear in the output .

