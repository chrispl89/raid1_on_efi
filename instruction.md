# Creating a RAID1 Array for EFI Partition

> ⚠️ WARNING: Setting up RAID1 for the EFI system partition is not officially supported by all Linux distributions or UEFI firmware. This guide is for advanced users who understand the risks. Always test in a virtual machine and create a full backup before proceeding.

This guide explains how to configure a RAID1 mirror for the `/boot/efi` partition, so that the system can continue booting even if one of the disks fails.

---

## Step-by-step Guide

### 1. Backup your current EFI partition
```bash
cp -r /boot/efi /boot/eficopy
```

---

### 2. Unmount the EFI partition
```bash
umount /boot/efi
```

---

### 3. Check available partitions
Use this to identify the correct EFI partitions:
```bash
fdisk -l
```

---

### 4. Create RAID1 array
> Replace `/dev/nvme0n1p1` and `/dev/nvme1n1p1` with the actual EFI partitions from step 3.

```bash
mdadm --create /dev/md3 --level=1 --raid-devices=2 --metadata=1.0 /dev/nvme0n1p1 /dev/nvme1n1p1
```

---

### 5. Monitor the status
```bash
cat /proc/mdstat
```

---

### 6. Format the new array with FAT32
```bash
mkfs.vfat /dev/md3
```

---

### 7. Update `/etc/fstab` with new UUID
Get the new UUID:
```bash
blkid
```
Replace the existing EFI UUID in `/etc/fstab` with the new one from `/dev/md3`.

---

### 8. Mount and restore EFI data
```bash
mount /boot/efi
cp -r /boot/eficopy/* /boot/efi/
```

---

### 9. Update EFI boot entries

#### View existing entries:
```bash
efibootmgr -v
```

#### Optionally remove old entry:
```bash
efibootmgr -B -b 0006
```

#### Add new boot entry:
```bash
efibootmgr --create --disk /dev/sdb --part 1 --label "debian2" --loader "\EFI\debian\shimx64.efi"
```

> Replace `/dev/sdb` and partition number with the correct device where RAIDed EFI is located.

---

### 🔄 Finalize with grub update
```bash
update-grub && update-grub2
```
