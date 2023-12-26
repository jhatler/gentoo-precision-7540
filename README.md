# Gentoo Setup on Dell Precision 7540

Goals:

- [ ] ([#4](https://github.com/jhatler/gentoo-precision-7540/issues/4)) Gentoo installed
- [X] ([#5](https://github.com/jhatler/gentoo-precision-7540/issues/5)) LUKS full disk encryption with performance tuning
- [X] ([#6](https://github.com/jhatler/gentoo-precision-7540/issues/6)) BTRFS raid0 for rootfs
- [X] ([#7](https://github.com/jhatler/gentoo-precision-7540/issues/7)) Striped LVM for VMs and other filesystems
- [X] ([#8](https://github.com/jhatler/gentoo-precision-7540/issues/8)) Ability to reallocate space between BTRFS and LVM
- [ ] ([#9](https://github.com/jhatler/gentoo-precision-7540/issues/9)) BTRFS snapshots for rollback integrated into emerge
- [ ] ([#10](https://github.com/jhatler/gentoo-precision-7540/issues/10)) System optimized with LTO, Graphite, PGO, and CFLAGS
- [ ] ([#11](https://github.com/jhatler/gentoo-precision-7540/issues/11)) systemd-boot configured with secure boot
- [ ] ([#12](https://github.com/jhatler/gentoo-precision-7540/issues/12)) Nvidia GPU working with CUDA and vGPU support
- [ ] ([#13](https://github.com/jhatler/gentoo-precision-7540/issues/13)) KVM configured with virt-manager and libvirt support
- [ ] ([#14](https://github.com/jhatler/gentoo-precision-7540/issues/14)) Podman setup for rootless containers
- [ ] ([#15](https://github.com/jhatler/gentoo-precision-7540/issues/15)) TMP2 support for unlocking LUKS
- [ ] ([#16](https://github.com/jhatler/gentoo-precision-7540/issues/16)) Yubikey support for unlocking LUKS
- [ ] ([#17](https://github.com/jhatler/gentoo-precision-7540/issues/17)) i3-gaps for window manager
- [ ] ([#18](https://github.com/jhatler/gentoo-precision-7540/issues/18)) spacemacs installed
- [ ] ([#19](https://github.com/jhatler/gentoo-precision-7540/issues/19)) vscode installed
- [ ] ([#20](https://github.com/jhatler/gentoo-precision-7540/issues/20)) Gentoo Containers for development via catalyst

## BIOS Setup

The BIOS was reset to BIOS Defaults and the following changes were made:

- System Configuration -> Integrated NIC
  - Enable UEFI Network Stack: Unchecked
  - Radio Option: Enabled
- System Configuration -> SATA Operation
  - Radio Option: AHCI
- System Configuration -> SMART Reporting
  - Enable SMART Reporting: Checked
- System Configuration -> USB Configuration
  - Enable USB Boot Support: Checked
  - Enable External USB Port: Checked
- System Configuration -> Thunderbolt Adapter Configuration
  - Thunderbolt: Checked
  - Thunderbolt Boot Support: Checked
  - Enable Thunderbolt (and PCIe behind TBT) Pre-boot Modules: Checked
  - Radio Option: No Security
- System Configuration -> USB PowerShare
  - Enable USB PowerShare: Checked
- Video -> Switchable Graphics
  - Enable Switchable Graphics: Checked
  - Discrete Graphics Controller: Unchecked
- Security -> UEFI Capsule Firmware Updates
  - Enable UEFI Capsule Firmware Updates: Checked
- Security -> TPM 2.0 Security
  - TPM On: Checked
  - Clear: Checked
  - Attestation Enable: Checked
  - Key Storage Enable: Checked
  - SHA-256: Checked
  - Radio Option: Enabled
- Security -> Absolute
  - Radio Option: Disabled
- Security -> SMM Security Mitigation
  - SMM Security Mitigation: Checked
- Secure Boot -> Secure Boot Enable
  - Secure Boot Enable: Checked
- Secure Boot -> Secure Boot Mode
  - Radio Option: Audit Mode
- Intel Software Guard Extensions -> Intel SGX Enable
  - Radio Option: Enabled
- Intel Software Guard Extensions -> Enclave Memory Size
  - Radio Option: 32 MB
- Maintenance -> BIOS Recovery
  - BIOS Recovery from Hard Drive: Unchecked
- SupportAssist System Resolution -> Auto OS Recovery Threshold
  - Radio Option: OFF
- SupportAssist System Resolution -> SupportAssist OS Recovery
  - SupportAssist OS Recovery: Unchecked
- SupportAssist System Resolution -> BIOSConnect
  - BIOSConnect: Unchecked

The system was then rebooted twice to allow the TPM to fully reset.

## Installation Environment Setup

The AMD64 LiveGUI USB Image was downloaded from here: [https://www.gentoo.org/downloads/](https://www.gentoo.org/downloads/)

[Rufus](https://rufus.ie/en/) was used to create a bootable USB drive from the image.

The USB drive was inserted into the laptop and the laptop was powered on.

F12 was pressed to enter the boot menu and the USB drive was selected.

The ```Boot LiveCD (kernel: gentoo)``` option was selected.

Konsole was opened.

A root shell was opened using ```sudo su``` and the Wi-Fi was connected to using ```nmtui```.

The gentoo repository was updated using the following commands:

```bash
emerge-webrsync
emerge --sync
```

Then the following commands were run to setup SSH access to allow for easier setup:

```bash
passwd gentoo
rc-update add sshd default
/etc/init.d/sshd start
```

## LUKS Performance testing

LUKS vs. Direct disk access was tested using FIO. 4MiB blocks were used to test the sequential read/write performance
and 4KiB blocks were used to test the random read/write performance. A single NVME disk was used for the tests. The 
results are summarized below:

| Test Type | Queues Enabled | Submit from Crypt CPUs | Jobs | Random IO | Read (MiB/s) | Write (MiB/s) |
|-----------|----------------|------------------------|------|-----------|--------------|---------------|
| Direct    | #N/A           | #N/A                   | 1    | FALSE     | 1414.305713  | 1412.039197   |
| Direct    | #N/A           | #N/A                   | 16   | FALSE     | 2712.619886  | 2721.282936   |
| Direct    | #N/A           | #N/A                   | 1    | TRUE      | 133.6726561  | 133.4808588   |
| Direct    | #N/A           | #N/A                   | 16   | TRUE      | 754.250453   | 754.6367817   |
| Encrypted | FALSE          | TRUE                   | 1    | FALSE     | 758.4574871  | 762.5904074   |
| Encrypted | FALSE          | FALSE                  | 1    | FALSE     | 665.4001265  | 674.0659266   |
| Encrypted | TRUE           | TRUE                   | 1    | FALSE     | 601.8466043  | 615.5794802   |
| Encrypted | TRUE           | FALSE                  | 1    | FALSE     | 498.8501911  | 515.51408     |
| Encrypted | FALSE          | TRUE                   | 16   | FALSE     | 2678.650202  | 2683.852674   |
| Encrypted | FALSE          | FALSE                  | 16   | FALSE     | 2657.468244  | 2670.798386   |
| Encrypted | TRUE           | TRUE                   | 16   | FALSE     | 2666.266011  | 2677.458804   |
| Encrypted | TRUE           | FALSE                  | 16   | FALSE     | 2646.007566  | 2655.478645   |
| Encrypted | FALSE          | TRUE                   | 1    | TRUE      | 112.1937857  | 112.0448322   |
| Encrypted | FALSE          | FALSE                  | 1    | TRUE      | 109.5733004  | 109.4292946   |
| Encrypted | TRUE           | TRUE                   | 1    | TRUE      | 100.8804913  | 100.6935177   |
| Encrypted | TRUE           | FALSE                  | 1    | TRUE      | 96.65823555  | 96.48545551   |
| Encrypted | FALSE          | TRUE                   | 16   | TRUE      | 750.838645   | 751.2814436   |
| Encrypted | FALSE          | FALSE                  | 16   | TRUE      | 753.6204519  | 754.012641    |
| Encrypted | TRUE           | TRUE                   | 16   | TRUE      | 850.8773651  | 851.1699362   |
| Encrypted | TRUE           | FALSE                  | 16   | TRUE      | 882.7599688  | 883.0642529   |

Based on these results, the following flags were selected for the LUKS setup:
- ```--perf-no_read_workqueue```
- ```--perf-no_write_workqueue```
- ```--perf-submit_from_crypt_cpus```

## Disk Setup

The NVME disks were erased using the following commands:

```bash
wipefs -fa /dev/nvme0n1
wipefs -fa /dev/nvme1n1
wipefs -fa /dev/nvme2n1
```

The desired disk layout is one that is partitioned identically on all three disks.
This allows for the disks to be used in RAID configurations with less complexity around certain calculations.

RAID1 is desired for the ESP partition. This will be unencrypted.

A separate RAID1 volume is desired for the /boot partition to make it easier to install other distros in the future.
The /boot partition will be encrypted using LUKS1. GRUB does not support LUKS2 well and this will ensure better
compatibility with other distros, if needed in the future.

There will be separate paritions for encrypted swap and the LVM volume group for the rest of the data. A LVM on LUKS
approach will be used to allow for the ability to resize the LVM volume group in the future. BTRFS will be used for
the root filesystem. RAID will be handled here at the BTRFS level so the filesystem has a better understanding of the
physical layout of the disks.

They disks were partitioned using the following commands:

```bash
for _disk in /dev/nvme{0,1,2}n1; do
  parted -a optimal "$_disk" --script mklabel gpt \
    mkpart primary 1MiB 2049MiB \
    mkpart primary 2049MiB 6145MiB \
    mkpart primary 6145MiB 14377MiB \
    mkpart primary 14377MiB 100% \
    set 1 esp on
done
```

In order for the ESP partitions to be readable by UEFI, the must use the 1.0 metadata format that places the metadata
at the end of the parition. The ESP RAID1 was setup using the following command:

```bash
mdadm --create /dev/md100 --level 1 --raid-disks 3 --metadata 1.0 /dev/nvme{0,1,2}n1p1
```

The /boot RAID1 was setup using the following commands:

```bash
mdadm --create /dev/md101 --level 1 --raid-disks 3 --metadata 1.0 /dev/nvme{0,1,2}n1p2
```

The /boot encrypted LUKS1 partition was setup and formatted using the following commands:

```bash
cryptsetup luksFormat --type luks1 /dev/md101
cryptsetup luksOpen /dev/md101 cryptboot
mkfs.ext4 -L boot /dev/mapper/cryptboot
```

The encrypted swap partitions were setup, formatted, and activated using the following commands:

```bash
cryptsetup luksFormat --type luks1 /dev/nvme0n1p3
cryptsetup luksFormat --type luks1 /dev/nvme1n1p3
cryptsetup luksFormat --type luks1 /dev/nvme2n1p3

cryptsetup luksOpen /dev/nvme0n1p3 cryptswap0
cryptsetup luksOpen /dev/nvme1n1p3 cryptswap1
cryptsetup luksOpen /dev/nvme2n1p3 cryptswap2

mkswap -L swap0 /dev/mapper/cryptswap0
mkswap -L swap1 /dev/mapper/cryptswap1
mkswap -L swap2 /dev/mapper/cryptswap2

swapon -L swap0
swapon -L swap1
swapon -L swap2
```

The encrypted LVM volume group was setup and formatted using the following commands:

```bash
cryptsetup luksFormat --type luks2 /dev/nvme0n1p4
cryptsetup luksFormat --type luks2 /dev/nvme1n1p4
cryptsetup luksFormat --type luks2 /dev/nvme2n1p4

cryptsetup luksOpen \
  --allow-discards \
  --perf-no_read_workqueue \
  --perf-no_write_workqueue \
  --perf-submit_from_crypt_cpus \
  /dev/nvme0n1p4 cryptdata0

cryptsetup luksOpen \
  --allow-discards \
  --perf-no_read_workqueue \
  --perf-no_write_workqueue \
  --perf-submit_from_crypt_cpus \
  /dev/nvme1n1p4 cryptdata1

cryptsetup luksOpen \
  --allow-discards \
  --perf-no_read_workqueue \
  --perf-no_write_workqueue \
  --perf-submit_from_crypt_cpus \
  /dev/nvme2n1p4 cryptdata2

pvcreate /dev/mapper/cryptdata0
pvcreate /dev/mapper/cryptdata1
pvcreate /dev/mapper/cryptdata2

vgcreate vg0 /dev/mapper/cryptdata{0,1,2}
```

The rootfs BTRFS RAID1 volume was setup and formatted using the following commands:

```bash
lvcreate -L 32G -n root0 vg0 /dev/mapper/cryptdata0
lvcreate -L 32G -n root1 vg0 /dev/mapper/cryptdata1
lvcreate -L 32G -n root2 vg0 /dev/mapper/cryptdata2
mkfs.btrfs -d raid1 -m raid1c3 --csum xxhash -n 4096 -L root /dev/mapper/vg0-root{0,1,2}
```

A thin pool was created for VM data using the following commands:

```bash
echo 'sys-fs/lvm2 thin' > /etc/portage/package.use/lvm2
emerge -av sys-fs/lvm2 sys-block/thin-provisioning-tools
lvcreate -L 768G -i3 -I4m -c 4m -Zn --thinpool thinpool0 vg0 /dev/mapper/cryptdata{0,1,2}
```

A BTRFS RAID0 volume was created for non-critical data using the following commands:

```bash
lvcreate -L 128G -n data0 vg0 /dev/mapper/cryptdata0
lvcreate -L 128G -n data1 vg0 /dev/mapper/cryptdata1
lvcreate -L 128G -n data2 vg0 /dev/mapper/cryptdata2
mkfs.btrfs -d raid0 -m raid1 --csum xxhash -n 4096 -L data /dev/mapper/vg0-data{0,1,2}
```

## Chroot Setup

BTRFS subvolumes were created for the rootfs, home, and select var directories using the following commands:

```bash
mkdir -p /mnt/btrfs/{root,data}
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed \
  /dev/mapper/vg0-root0 /mnt/btrfs/root
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed \
  /dev/mapper/vg0-data0 /mnt/btrfs/data

btrfs subvolume create /mnt/btrfs/root/@gentoo
btrfs subvolume create /mnt/btrfs/data/@gentoo-home
btrfs subvolume create /mnt/btrfs/data/@gentoo-var-cache
btrfs subvolume create /mnt/btrfs/data/@gentoo-var-db
btrfs subvolume create /mnt/btrfs/data/@gentoo-var-log
btrfs subvolume create /mnt/btrfs/data/@gentoo-var-tmp
```

The chroot was mounted using the following commands:

```bash
mkdir /mnt/chroot
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed,subvol=@gentoo \
  /dev/mapper/vg0-root0 /mnt/chroot

mkdir /mnt/chroot/boot
mount /dev/mapper/cryptboot /mnt/chroot/boot

mkdir /mnt/chroot/boot/efi
mount /dev/md100 /mnt/chroot/boot/efi

mkdir /mnt/chroot/home
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed,subvol=@gentoo-home \
  /dev/mapper/vg0-data0 /mnt/chroot/home

mkdir -p /mnt/chroot/var/{cache,db,log,tmp}
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed,subvol=@gentoo-var-cache \
  /dev/mapper/vg0-data0 /mnt/chroot/var/cache
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed,subvol=@gentoo-var-db \
  /dev/mapper/vg0-data0 /mnt/chroot/var/db
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed,subvol=@gentoo-var-log \
  /dev/mapper/vg0-data0 /mnt/chroot/var/log
mount -t btrfs \
  -o compress=zstd,discard=async,max_inline=0,space_cache=v2,ssd,commit=120,user_subvol_rm_allowed,subvol=@gentoo-var-tmp \
  /dev/mapper/vg0-data0 /mnt/chroot/var/tmp
```

### Stage 3 Install

The ```systemd | merged-usr``` stage3 tarball link was copied from here:
[https://www.gentoo.org/downloads/](https://www.gentoo.org/downloads/)

The stage3 tarball was downloaded and verified using the following commands:

```bash
cd /mnt/chroot
wget ${STAGE3_URL}
wget ${STAGE3_URL}.CONTENTS.gz
wget ${STAGE3_URL}.DIGESTS
wget ${STAGE3_URL}.sha256
wget ${STAGE3_URL}.asc

sha256sum --check stage3-amd64-systemd-mergedusr-*.tar.xz.sha256

openssl dgst -r -sha512 stage3-amd64-systemd-mergedusr-*.tar.xz
openssl dgst -r -sha512 stage3-amd64-systemd-mergedusr-*.tar.xz.CONTENTS.gz
openssl dgst -r -blake2b512 stage3-amd64-systemd-mergedusr-*.tar.xz
openssl dgst -r -blake2b512 stage3-amd64-systemd-mergedusr-*.tar.xz.CONTENTS.gz

# compare above output to hashes in this file
cat stage3-amd64-systemd-mergedusr-*.tar.xz.DIGESTS

gpg --import /usr/share/openpgp-keys/gentoo-release.asc
gpg --verify stage3-amd64-systemd-mergedusr-*.tar.xz.asc
gpg --verify stage3-amd64-systemd-mergedusr-*.tar.xz.DIGESTS
gpg --verify stage3-amd64-systemd-mergedusr-*.tar.xz.sha256
```

The stage3 tarball was extracted using the following command:

```bash
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

### System Mounts

The mounts needed to open a chroot were setup using the following commands:

```bash
mount --types proc /proc /mnt/chroot/proc
mount --rbind /sys /mnt/chroot/sys
mount --make-rslave /mnt/chroot/sys
mount --rbind /dev /mnt/chroot/dev
mount --make-rslave /mnt/chroot/dev
mount --bind /run /mnt/chroot/run
mount --make-slave /mnt/chroot/run
```

### DNS Setup

The chroot DNS was setup using the following commands:

```bash
cp --dereference /etc/resolv.conf /mnt/chroot/etc/
```


## Bootstrapping

Bootstrapping the system consists of installing the portage tree, configuring portage, setting the profile, and
updating the system, and the rebuilding everything twice to ensure all packages are built with the latest compiler.

After the initial update is completed, the GentooLTO overlay will be used as a reference to enable LTO and other
optimizations. Doing this before the bootstrap will minimize the number of world rebuilds needed.
Package testing needs enabled to ensure there are no issues with the packages being built.

ccache will also be setup to speed up rebuilds.

### Locale Setup

The following lines were uncommented in ```/mnt/chroot/etc/locale.gen```:

```text
en_US ISO-8859-1
en_US.UTF-8 UTF-8
```

Then ```locale-gen``` was run within the chroot to generate the locales.

### Portage Tree

Mirrors were selected using the following command:

```bash
mirrorselect -i -o >> /mnt/chroot/etc/portage/make.conf
```

The portage tree was configured using the following commands:

```bash
mkdir --parents /mnt/chroot/etc/portage/repos.conf
cp /mnt/chroot/usr/share/portage/config/repos.conf /mnt/chroot/etc/portage/repos.conf/gentoo.conf
```

The portage tree was synced using the following commands:

```bash
chroot /mnt/chroot
emerge-webrsync
emerge --sync
```

### Portage Configuration

The ```/etc/portage/make.conf``` file was updated in the chroot to the following:

```bash

## LTO Settings

## Use Flags
USE=""

## Compilter Settings
COMMON_FLAGS="-march=skylake -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"

CHOST="x86_64-pc-linux-gnu"

CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3"

CMAKE_MAKEFILE_GENERATOR=ninja

MAKEOPTS="-j20"

## Directory Settings
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"

## Mirror List
GENTOO_MIRRORS="https://mirrors.rit.edu/gentoo/"

## Locale Settings
LC_MESSAGES=C.utf8
L10N="en"

## ACCEPT Settings
ACCEPT_KEYWORDS="amd64"
ACCEPT_LICENSE="-* @FREE"

## Logging
PORTAGE_ELOG_CLASSES="info warn error log qa"
PORTAGE_ELOG_SYSTEM="echo save"

## Portage Features
FEATURES="binpkg-logs binpkg-multi-instance buildpkg candy ccache parallel-fetch parallel-install preserve-libs split-elog split-log test unmerge-logs"
PORTAGE_NICENESS=15
CCACHE_DIR=/var/cache/ccache
```

### Profile

The ```default/linux/amd64/17.1/systemd/merged-usr``` profile was selected using the following command:

```bash
eselect profile set default/linux/amd64/17.1/systemd/merged-usr
```

### System Update

The system was updated using the following commands:

```bash
emerge -avuDU --with-bdeps=y --jobs=32 --load-average=20 @world
emerge -av --depclean
```

### ccache Setup

ccache was installed using the following command:

```bash
emerge -av dev-util/ccache
```

The ```ccache``` feature was added to ```FEATURES``` in ```/etc/portage/make.conf```. The ```CCACHE_DIR``` variable
was set to ```/var/cache/ccache``` in ```/etc/portage/make.conf``` as well.

The ccache cache directory was configured using the following commands:

```bash
mkdir -p /mnt/chroot/var/cache/ccache
chown -R portage:portage /mnt/chroot/var/cache/ccache
cat <<EOF > /mnt/chroot/var/cache/ccache/ccache.conf
max_size = 64.0G
umask = 002
hash_dir = false
compiler_check = %compiler% -dumpversion
cache_dir_levels = 4
EOF
```

