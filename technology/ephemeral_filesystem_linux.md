# Ephemeral File Systems in Linux

Comprehensive guide to Linux filesystems that exist only in memory or are
automatically created/destroyed — tmpfs, ramfs, overlayfs, initramfs,
cloud/container ephemeral storage, performance, security, and zram/zswap.
**Author:** Jack Liu Shurui | **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)

---
## 1. Introduction
**Ephemeral filesystems** store data only in RAM or as auto-created/destroyed
layers — no persistent trace remains on disk after shutdown, unmount, or
container teardown.
| Type | Backing Store | Key Use Case |
|------|--------------|-------------|
| tmpfs | RAM + swap | `/tmp`, `/dev/shm`, container mounts |
| ramfs | RAM (pinned, no swap) | Crypto keys, sensitive data |
| overlayfs | Disk (upperdir) | Docker/container writable layers |
| initramfs | RAM (cpio archive) | Early boot userspace |
| zram | Compressed RAM | Swap-on-compressed-RAM |

---

## 2. tmpfs — RAM-Backed Temporary Filesystem

`tmpfs` (Linux 2.4+) stores data in virtual memory. Default for `/tmp`,
`/dev/shm`, `/run`. Configurable size limits (default 50% RAM), swap backing
(pages can page out), POSIX compliant, visible in `df -h`. Lost on reboot.

```
mount -t tmpfs -o size=1G,nr_inodes=10k,mode=1777 tmpfs /mnt/tmp

# /etc/fstab
tmpfs /mnt/tmp tmpfs rw,size=2G,nr_inodes=5k,noexec,nodev,nosuid 0 0
```

| Option | Description |
|--------|-------------|
| `size=` | Max bytes (k/m/g/% suffixes). Default: 50% RAM. |
| `nr_inodes=` | Max inodes. Default: half physical RAM pages. |
| `mode=` | Root dir permissions (octal). |
| `uid=`/`gid=` | Owner/group of mount root. |

Resize live: `mount -o remount,size=4G /dev/shm`

Common mounts: **`/run`** (PID files, sockets), **`/dev/shm`** (POSIX shared
memory, 50% RAM), **`/tmp`** (tmpfs on Fedora/Arch/OpenSUSE).

**Limitation**: Swap backing means "RAM data" touches disk under pressure. Not
a block device — unformattable with ext4/XFS.

---

## 3. ramfs — Simple RAM Filesystem

`ramfs` (pre-dates tmpfs) exports page/dentry caches as a RAM fs. No size
limit (grows until OOM), no swap backing (pages pinned), hidden from `df -h`,
no page reclaim.

```
mount -t ramfs ramfs /mnt/ram
mount -t ramfs -o size=512M ramfs /mnt/ram   # VFS soft hint only
```

Use for: crypto keys (never swap), embedded systems (known budget), Oracle SGA
(historical). **Caution**: can consume all RAM → OOM. Prefer tmpfs with
`size=` unless no-swap semantics required.

### tmpfs vs ramfs

| Feature | tmpfs | ramfs |
|---------|-------|-------|
| Size limit | Configurable (default 50%) | None (OOM risk) |
| Swap backing | Yes | No (pinned) |
| df visibility | Yes | No |
| Performance | Identical (same page cache) | Identical |

---

## 4. overlayfs — Union Mount Filesystem

### What It Is

`overlayfs` merges multiple directories ("layers") into one unified view. The
storage backbone of Docker, Podman, and Kubernetes.

- **Lowerdir** (read-only): Base image layers.
- **Upperdir** (writable): All modifications — **ephemeral**.
- **Workdir**: Empty dir on same fs as upperdir, for internal bookkeeping.
- **Merged**: The mount point.

### How It Works

Read: file in upperdir → serve from upperdir. Otherwise → fall through to
lowerdir. **Copy-up**: First write to a lower-layer file copies it to upperdir,
then modification applies there. **Whiteout**: Deleted lower-layer files are
hidden by whiteout entries in upperdir.

### Manual Mount

```
mkdir -p /overlay/{lower,upper,work,merged}
echo "base" > /overlay/lower/base.txt

mount -t overlay overlay \
  -o lowerdir=/overlay/lower,upperdir=/overlay/upper,workdir=/overlay/work \
  /overlay/merged

echo "new" > /overlay/merged/new.txt   # Goes to upperdir
umount /overlay/merged
```

Multiple lower dirs (Linux 4.0+):
```
mount -t overlay overlay -o lowerdir=/l3:/l2:/l1,upperdir=/u,workdir=/w /m
```

### 4.4 Docker overlay2

```
/var/lib/docker/overlay2/
├── <layer-hash>/        # Image layer (lower)
├── <container-id>/      # Writable layer (upper) — ephemeral
│   ├── diff/
│   └── work/
└── merged/
```

Container's upperdir is **discarded on `docker rm`** — the mechanism behind
container "statelessness."

### 4.5 overlayfs vs tmpfs in Containers

| Aspect | overlayfs upperdir | tmpfs mount |
|--------|-------------------|-------------|
| Backing | Disk (ext4/xfs) | RAM (+ swap) |
| Speed | Disk-bound | RAM-bound (10-100× faster) |
| Persist restarts? | Yes (until rm) | No |
| Use case | Image writes | Runtime temp, caches |

---

## 5. Ephemeral Root Filesystems

### 5.1 initramfs

Loaded at boot: bootloader loads kernel + compressed cpio archive → kernel
extracts into tmpfs-based `rootfs` → executes `/init` → drivers load, real
root mounted → `switch_root()` → initramfs discarded.

| Aspect | initramfs | initrd (legacy) |
|--------|-----------|-----------------|
| Format | Compressed cpio | Loopback block device image |
| Kernel support | Built-in cpio unpacker | Needs fs driver in kernel |
| Memory | Efficient (only used pages) | Whole image loaded |
| Since | Linux 2.5+ | Pre-2.5 (deprecated) |

### 5.2 Dracut

Builds initramfs on Fedora/RHEL:

```
dracut                                    # Current kernel
dracut --kver 6.7.5-200.fc39.x86_64      # Specific kernel
dracut --regenerate-all                   # All installed kernels
lsinitrd /boot/initramfs-$(uname -r).img  # List contents
dracut --list-modules                     # Available modules
```

Organized into modules (disk decrypt, LVM, mdadm, network root).

### 5.3 Memdisk (Syslinux)

Loads a full disk image into RAM as an emulated BIOS disk — can partition and
format it. Used for PXE-booted live CDs and diskless systems.

```
# syslinux.cfg
LABEL memdisk-test LINUX memdisk INITRD myimage.img APPEND iso
```

### 5.4 rootfs

Special tmpfs instance the kernel mounts as initial root. Receives unpacked
initramfs. Cannot be unmounted. Replaced via `switch_root()`.

---

## 6. Cloud & Container Ephemeral Storage

### 6.1 Docker tmpfs Mounts

```
docker run --tmpfs /app/tmp:noexec,size=100m nginx
docker run --mount type=tmpfs,destination=/cache,tmpfs-size=1G nginx

# docker-compose.yml
services:
  app:
    image: nginx
    tmpfs:
      - /app/tmp:noexec,size=100m
```

Use for: high-speed temp storage, avoiding overlay2 writable layer bloat.

### 6.2 Kubernetes emptyDir with tmpfs

```yaml
volumes:
- name: ram-storage
  emptyDir:
    medium: Memory        # Creates tmpfs mount
    sizeLimit: 512Mi      # K8s 1.20+; node allocatable if omitted
```

- Data lost when Pod is deleted.
- Shared across all containers in the Pod.
- No support for StorageClass, snapshots, or resize.

### 6.3 Generic Ephemeral Volumes (K8s 1.23+)

```yaml
volumes:
- name: scratch
  ephemeral:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        resources: { requests: { storage: 1Gi } }
```

Supports StorageClass, snapshots, resize, and any CSI driver.

### 6.4 AWS Instance Store

Physically attached NVMe SSDs on c5d/m5d/r5d/i3/i4i instances.

```
lsblk | grep nvme
mkfs.ext4 /dev/nvme1n1 && mount /dev/nvme1n1 /mnt/ephemeral
```

- Direct NVMe (not network-attached) → very high IOPS.
- Data lost on instance stop/terminate. No built-in replication.
- Use: caches, scratch, Spark shuffle.

### 6.5 GCP Local SSDs

Physically attached NVMe (up to 3 TB/VM). ~900K read IOPS, ~700K write IOPS.

```
mkfs.ext4 -F /dev/nvme0n1
mount -o discard,defaults /dev/nvme0n1 /mnt/localssd
```

Ephemeral — data persists only while instance runs.

---

## 7. Practical Use Cases

### 7.1 Container Storage

| Scenario | Approach |
|----------|----------|
| Writable layer | overlayfs upperdir (default) |
| Runtime temps | tmpfs mount (`--tmpfs`) |
| Shared cache | emptyDir `medium: Memory` |
| Sensitive data | Docker tmpfs with `noexec` |

### 7.2 CI/CD Workspaces

tmpfs builds can be 5-10× faster for I/O-bound steps:

```
mount -t tmpfs -o size=10G tmpfs /var/lib/jenkins/workspace
```

Benefits: clean builds, no stale files, reduced SSD wear.

### 7.3 Caches and Build Artifacts

```
mount -t tmpfs -o size=2G tmpfs /var/cache/apt
mount -t tmpfs -o size=5G tmpfs /home/user/.ccache
mount -t tmpfs -o size=4G tmpfs /home/user/.m2
mount -t tmpfs -o size=2G tmpfs /home/user/.npm
mount -t tmpfs -o size=1G,uid=1000,gid=1000 tmpfs /home/user/.cache/chromium
```

### 7.4 Testing

- Database test data on tmpfs → eliminates disk I/O variability.
- overlayfs for snapshot/rollback in destructive fs tests.
- Rapid prototyping without filling disk.

---

## 8. Performance & Benchmarking

### 8.1 RAM vs Disk Magnitudes

| Metric | RAM (tmpfs) | NVMe SSD | SATA SSD | HDD |
|--------|-------------|----------|----------|-----|
| Read latency | ~100 ns | ~50-100 µs | ~100-200 µs | ~5-10 ms |
| Seq. read | ~15-25 GB/s | ~3-7 GB/s | ~500 MB/s | ~150 MB/s |
| Random 4K read IOPS | ~10M+ | ~500K-1M | ~80K | ~200 |

### 8.2 Benchmarking with fio

```
# Install: sudo apt install fio

# Sequential read
fio --name=seqread --size=1G --rw=read --bs=1M --direct=0 \
    --directory=/mnt/tmpfs --numjobs=4 --group_reporting

# Random 4K read IOPS
fio --name=randread --size=1G --rw=randread --bs=4k --direct=0 \
    --directory=/mnt/tmpfs --numjobs=8 --iodepth=64 --group_reporting

# Random 4K write IOPS
fio --name=randwrite --size=1G --rw=randwrite --bs=4k --direct=0 \
    --directory=/mnt/tmpfs --numjobs=8 --iodepth=64 --group_reporting

# Mixed 70/30
fio --name=rwmix --size=1G --rw=rw --rwmixread=70 --bs=4k --direct=0 \
    --directory=/mnt/tmpfs --numjobs=4 --iodepth=32 --group_reporting

# Latency (99th %ile)
fio --name=latency --size=512M --rw=randread --bs=4k --direct=0 \
    --directory=/mnt/tmpfs --lat_percentiles=1 \
    --percentile_list=50:90:99:99.9
```

Expected: tmpfs ~20 GB/s seq, ~8-10M IOPS random 4K. NVMe ~5 GB/s seq, ~800K
IOPS. tmpfs `--direct=1` has no effect (always uses page cache).

---

## 9. Security Considerations

### 9.1 Non-Persistence

Data gone on: `umount`, `reboot`, power loss / `docker rm`.

**Risk**: Live attacker can read tmpfs data. Mitigate with permissions:
```
mount -t tmpfs -o size=1G,mode=0700,noexec,nodev,nosuid tmpfs /mnt/tmp
```

### 9.2 Data Isolation

- tmpfs: per-mount instances. Separate containers get separate instances.
- overlayfs: per-container upperdir. Deleted on `docker rm`.
- `/dev/shm`: per IPC namespace. Docker gives each container its own.

### 9.3 DoS Prevention

Without limits, ramfs/tmpfs can fill RAM:
```
mount -t tmpfs -o size=100M,nr_inodes=10k tmpfs /mnt/tmp
docker run --memory=512m --tmpfs /tmp:size=100m alpine
```

### 9.4 Swap Leakage

tmpfs pages can swap out, leaking sensitive data to disk:
```
swapoff -a                           # Disable swap
mount -t ramfs ramfs /mnt/secure     # Never swaps
```
Or use `mlock()` to pin pages in application code.

### 9.5 Cgroup Enforcement

Container memory limits trigger tmpfs page reclamation from that container's
cgroup when limits are hit.

---

## 10. zram & zswap — Compressed RAM Devices

### 10.1 zram

Compressed block device in RAM. Commonly used as swap (2:1-4:1 compression).

```
modprobe zram
zramctl --find --size 4G --algorithm zstd
mkswap /dev/zram0 && swapon /dev/zram0 -p 100

# As filesystem
mkfs.ext4 /dev/zram0 && mount /dev/zram0 /mnt/zram

# Monitor
zramctl
cat /sys/block/zram0/mm_stat
```

**Use**: IoT, Raspberry Pi, Chromebooks, Android, Proxmox (default swap).

### 10.2 zswap

Compressed write-back cache sitting between page-out and physical swap disk.
Reduces disk I/O for swapped pages.

```
echo 1 > /sys/module/zswap/parameters/enabled
echo zstd > /sys/module/zswap/parameters/compressor
echo 20 > /sys/module/zswap/parameters/max_pool_percent
```

| Feature | zram | zswap |
|---------|------|-------|
| Role | Compressed block device (acts as swap) | Cache between RAM and swap disk |
| Needs disk swap? | No | Yes |
| Best for | Low-RAM devices | Systems with RAM + swap disk |

### 10.3 Compression Algorithms

| Algo | Speed | Ratio | Since |
|------|-------|-------|-------|
| lz4 | Fastest | Low | Built-in |
| zstd | Fast | Good | 4.18 |
| lzo | Very fast | Low | Built-in |

Recommend: **lz4** for speed, **zstd** for balance.

### 10.4 zram-generator (systemd)

```ini
# /etc/systemd/zram-generator.conf
[zram0]
zram-size = ram * 2
compression-algorithm = zstd
swap-priority = 100
```

---

## 11. Practical Commands

### 11.1 tmpfs

```
sudo mount -t tmpfs -o size=1G tmpfs /mnt/fast
sudo mount -o remount,size=2G /mnt/fast
df -h -t tmpfs
echo "tmpfs /mnt/fast tmpfs rw,size=2G,noexec,nodev,nosuid 0 0" | sudo tee -a /etc/fstab
```

### 11.2 Docker tmpfs

```
docker run --rm --tmpfs /tmp:noexec,size=100m alpine dd if=/dev/zero of=/tmp/test bs=1M count=50
docker run --rm --mount type=tmpfs,destination=/cache,tmpfs-size=500m alpine df -h /cache
```

### 11.3 Kubernetes emptyDir

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-cache
spec:
  containers:
  - name: app
    image: alpine
    command: ["/bin/sh", "-c", "while true; do date >> /cache/log; sleep 5; done"]
    volumeMounts:
    - name: ram
      mountPath: /cache
  volumes:
  - name: ram
    emptyDir:
      medium: Memory
      sizeLimit: 256Mi
```

### 11.4 overlayfs Demo

```bash
#!/bin/bash
D=$(mktemp -d); mkdir -p $D/{l,u,w,m}
echo "base" > $D/l/base.txt
sudo mount -t overlay overlay -o lowerdir=$D/l,upperdir=$D/u,workdir=$D/w $D/m
echo "upper" > $D/m/new.txt
cat $D/u/new.txt
sudo umount $D/m
rm -rf $D
```

### 11.5 zram

```
sudo modprobe zram
echo lz4 | sudo tee /sys/block/zram0/comp_algorithm
echo 2G | sudo tee /sys/block/zram0/disksize
sudo mkswap /dev/zram0 && sudo swapon /dev/zram0 -p 100
zramctl && swapon --show
```

### 11.6 Systemd Mount Unit

```ini
# /etc/systemd/system/mnt-ephemeral.mount
[Unit] Description=Ephemeral tmpfs
[Mount] What=tmpfs Where=/mnt/ephemeral Type=tmpfs
Options=rw,size=2G,nr_inodes=50k,noexec,nodev,nosuid,mode=0755
[Install] WantedBy=multi-user.target
```
```
sudo systemctl enable --now mnt-ephemeral.mount
```

---

## 12. Comparison Table

| Filesystem | Backing | Size Limit | Swap-back | Persistence | Primary Use |
|------------|---------|------------|-----------|-------------|-------------|
| **tmpfs** | RAM (+ swap) | Yes (50% default) | Yes | None | `/tmp`, `/dev/shm`, caches |
| **ramfs** | RAM (pinned) | No (OOM risk) | No | None | Crypto keys, no-swap |
| **overlayfs** | Disk (upperdir) | Disk free | N/A | Until `rm` | Container writable layers |
| **initramfs** | RAM (cpio) | Archive size | N/A | Until boot done | Early boot rootfs |
| **zram** | Compressed RAM | Configurable | N/A (is swap) | None | Compressed swap |
| **zswap** | RAM + disk | % of RAM | Backs disk swap | None | Write-back swap cache |
| **Docker tmpfs** | RAM (+ swap) | Yes (size flag) | Yes | With container | Container temps |
| **K8s emptyDir** | Disk or RAM | Optional (v1.20+) | Depends | With Pod | Pod-scoped temps |
| **AWS Inst. Store** | Local NVMe | Device capacity | N/A | With instance | Cloud scratch |
| **GCP Local SSD** | Local NVMe | Device capacity | N/A | With instance | Cloud scratch |
| **Memdisk** | RAM (image) | Image size | N/A | Until boot | Legacy/ISO boot |

---

## 13. References

1. kernel.org tmpfs: https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html
2. kernel.org ramfs/rootfs/initramfs: https://www.kernel.org/doc/html/latest/filesystems/ramfs-rootfs-initramfs.html
3. kernel.org overlayfs: https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html
4. Docker tmpfs: https://docs.docker.com/engine/storage/tmpfs/
5. Docker overlay2: https://docs.docker.com/engine/storage/drivers/overlayfs-driver/
6. K8s Volumes: https://kubernetes.io/docs/concepts/storage/volumes/
7. K8s Ephemeral Volumes: https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/
8. dracut(8): https://www.man7.org/linux/man-pages/man8/dracut.8.html
9. AWS Instance Store: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html
10. GCP Local SSD: https://cloud.google.com/compute/docs/disks/local-ssd
11. ArchWiki tmpfs: https://wiki.archlinux.org/title/Tmpfs
12. ArchWiki zram: https://wiki.archlinux.org/title/Zram
13. fio: https://fio.readthedocs.io/
