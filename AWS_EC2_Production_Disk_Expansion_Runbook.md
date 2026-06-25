# AWS EC2 Production Disk Expansion Runbook

> Version: 1.0\
> Applicable to: AWS EC2 Linux Instances

## Supported Operating Systems

  OS                   Filesystem   Online Supported
  -------------------- ------------ ------------------
  RHEL 7/8/9           XFS          ✅
  Rocky Linux          XFS          ✅
  AlmaLinux            XFS          ✅
  Oracle Linux         XFS          ✅
  Amazon Linux 2       XFS / EXT4   ✅
  Amazon Linux 2023    XFS          ✅
  Ubuntu 18/20/22/24   EXT4         ✅
  Debian               EXT4         ✅
  SUSE                 Btrfs/XFS    ✅

------------------------------------------------------------------------

# Workflow

1.  Create EBS Snapshot
2.  Modify EBS Volume
3.  Verify OS detects new size
4.  Expand Partition
5.  Expand Filesystem
6.  Validate
7.  Application Health Check

------------------------------------------------------------------------

# Identify Current Layout

``` bash
hostnamectl
lsblk
lsblk -f
df -h
df -Th
fdisk -l
blkid
```

For LVM:

``` bash
pvs
vgs
lvs
```

------------------------------------------------------------------------

# Create Snapshot

AWS Console

EC2 → Volumes → Select Volume → Actions → Create Snapshot

Never skip this step in Production.

------------------------------------------------------------------------

# Modify Volume

AWS Console

EC2 → Volumes

Actions → Modify Volume

Example

50 GB → 60 GB

Wait until Optimizing or Completed.

------------------------------------------------------------------------

# Verify

``` bash
lsblk
fdisk -l
```

Disk should increase but partition remains unchanged.

------------------------------------------------------------------------

# Install growpart

## RHEL

``` bash
dnf install -y cloud-utils-growpart
```

or

``` bash
yum install -y cloud-utils-growpart
```

## Ubuntu

``` bash
apt update
apt install -y cloud-guest-utils
```

------------------------------------------------------------------------

# Grow Partition

NVMe example

``` bash
growpart /dev/nvme0n1 4
```

Xen example

``` bash
growpart /dev/xvda 1
```

------------------------------------------------------------------------

# Filesystem Expansion

## XFS

``` bash
xfs_growfs /
```

## EXT4

``` bash
resize2fs /dev/nvme0n1p4
```

## Btrfs

``` bash
btrfs filesystem resize max /
```

------------------------------------------------------------------------

# LVM Procedure

``` bash
growpart /dev/nvme0n1 2
pvresize /dev/nvme0n1p2
lvextend -l +100%FREE /dev/rhel/root
xfs_growfs /
```

For EXT4

``` bash
resize2fs /dev/rhel/root
```

------------------------------------------------------------------------

# Validation

``` bash
lsblk
df -h
df -Th
blkid
mount
dmesg | tail -20
```

Application Validation

-   Verify services
-   Verify logs
-   Verify monitoring
-   Verify disk alerts

------------------------------------------------------------------------

# Rollback

1.  Stop change
2.  Restore EBS Snapshot
3.  Replace Root Volume
4.  Boot Instance
5.  Validate

------------------------------------------------------------------------

# Troubleshooting

## growpart: NOCHANGE

Partition already occupies available space.

## xfs_growfs: not mounted

Run against mounted filesystem.

## resize2fs: Device busy

Usually safe on mounted EXT4. Verify correct device.

## Device not found

Run

``` bash
lsblk
nvme list
```

------------------------------------------------------------------------

# Device Naming

Nitro

``` text
/dev/nvme0n1
/dev/nvme1n1
```

Older Xen

``` text
/dev/xvda
/dev/xvdf
```

------------------------------------------------------------------------

# Production Checklist

-   Snapshot completed
-   Change approved
-   Monitoring enabled
-   Backup verified
-   Volume modified
-   Partition grown
-   Filesystem grown
-   Validation completed
-   Application healthy
-   Documentation updated

------------------------------------------------------------------------

# Common Commands

``` bash
lsblk
lsblk -f
df -h
df -Th
blkid
fdisk -l
growpart --help
xfs_growfs /
resize2fs <device>
pvresize <pv>
lvextend -l +100%FREE <lv>
pvs
vgs
lvs
```
