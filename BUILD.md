# Build Instructions

This image must be built on **Linux**. The build uses `losetup`, `sfdisk`, `mkfs.vfat`, `mkfs.ext4`, and `mount`, which are Linux-only tools. On macOS or Windows, use a VM or Docker container running Ubuntu/Debian.

---

## 1. Install dependencies

```bash
sudo apt-get update
sudo apt-get install -y \
    git \
    wget \
    bc \
    xz-utils \
    gzip \
    p7zip-full \
    squashfs-tools \
    dosfstools \
    e2fsprogs \
    util-linux \
    sfdisk
```

---

## 2. Clone the repository with submodules

```bash
git clone --recursive https://github.com/eduardofilo/RG350_adam_image.git
cd RG350_adam_image
```

If you already cloned without `--recursive`:

```bash
git submodule update --init
```

---

## 3. (Optional) Tune build parameters

Open `build.sh` and adjust the parameters at the top if needed:

| Parameter | Default | Notes |
|---|---|---|
| `MAKE_PGv1` | `true` | Build image for GCW-Zero / PocketGo2 v1 |
| `MAKE_RG` | `true` | Build image for RG350 / RG280 / RG300X |
| `COMP` | `xz` | Compression: `gz` (faster) or `xz` (smaller) |
| `SIZE_M` | `3200` | Final image size in MiB |

To build only the RG350 image (skip PocketGo2/GCW-Zero):

```bash
sed -i 's/^MAKE_PGv1=true/MAKE_PGv1=false/' build.sh
```

---

## 4. Build

The script requires root for loop device and mount operations. It will call `sudo` automatically if not already root.

```bash
./build.sh
```

The build will:
1. Download `gcw0-update-2022-09-22.opk` (ODbeta kernel/rootfs) from GitHub Releases
2. Download `RetroArch 1.22.2` for `mips32-odbeta` from the libretro stable buildbot
3. Assemble a 3200 MiB image with two partitions (FAT32 boot + ext4 data)
4. Compress the result with xz

---

## 5. Output

Finished images are written to the `releases/` directory:

| File | Device |
|---|---|
| `adam_v2.2_PGv1.img.xz` | GCW-Zero, PocketGo2 v1 |
| `adam_v2.2.img.xz` | RG350, RG350M, RG350P, RG280V, RG280M, RG300X, PocketGo2 v2 |

Flash to the internal microSD with:

```bash
xz -d adam_v2.2.img.xz
sudo dd if=adam_v2.2.img of=/dev/sdX bs=4M status=progress conv=fsync
```

Replace `/dev/sdX` with your card's device node (verify with `lsblk` before running).
