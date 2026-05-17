# QEMU Guide for Firmware Hacking

## Two types of QEMU

### System mode (qemu-system-*) - emulates a full machine
Boots an entire OS with kernel, network, filesystem - like running a virtual router on your desk.

| Binary | What it emulates |
|--------|-----------------|
| qemu-system-arm | Full ARM machine (most SOHO routers: TP-Link, Netgear, Ubiquiti) |
| qemu-system-mipsel | Full MIPS little-endian machine (older Linksys, D-Link, some Netgear) |

### User mode (qemu-*-static) - runs a single binary
No kernel, no OS - just runs one ARM/MIPS binary on your x86 machine. Much faster for quick testing.

| Binary | What it does |
|--------|-------------|
| qemu-arm-static | Run a single ARM binary (e.g., the router's httpd) |
| qemu-mipsel-static | Run a single MIPS binary |

---

## Practical examples

### 1. User mode - run a single router binary

Say you extracted a router firmware with binwalk and found /usr/sbin/httpd:

```bash
# Check what architecture the binary is
file ./httpd
# output: ELF 32-bit LSB executable, ARM, ...

# Run it directly on your x86 PC
qemu-arm-static ./httpd

# If it needs libraries from the firmware, point to them
qemu-arm-static -L ./squashfs-root/ ./squashfs-root/usr/sbin/httpd
```

`-L` tells QEMU to use the firmware's own /lib for shared libraries instead of your host's.

### 2. User mode - debug a binary with GDB

This is where you find the bugs:

```bash
# Start the binary paused, waiting for GDB on port 1234
qemu-arm-static -g 1234 -L ./squashfs-root/ ./squashfs-root/usr/sbin/httpd &

# In another terminal, attach GDB
gdb-multiarch -q ./squashfs-root/usr/sbin/httpd
(gdb) target remote :1234
(gdb) break main
(gdb) continue
```

Now you can step through ARM code, set breakpoints, inspect memory - all on your x86 PC.

### 3. System mode - boot an entire firmware

For testing the full exploit chain (web UI + network + services together):

```bash
qemu-system-arm \
  -M virt \
  -kernel zImage \
  -drive file=rootfs.ext4,format=raw \
  -append "root=/dev/vda console=ttyAMA0" \
  -nographic \
  -net nic -net user,hostfwd=tcp::8080-:80
```

Then browse http://localhost:8080 to hit the router's web UI running in the emulator.

### 4. System mode with MIPS (typical for older routers)

```bash
qemu-system-mipsel \
  -M malta \
  -kernel vmlinux-malta \
  -hda rootfs.qcow2 \
  -append "root=/dev/sda1" \
  -nographic \
  -net nic -net user,hostfwd=tcp::8080-:80
```

---

## The Pwn2Own workflow with QEMU

```
1. Download firmware from vendor site
        |
2. binwalk -e firmware.bin        -> extracts squashfs-root/
        |
3. file squashfs-root/usr/sbin/httpd   -> tells you ARM or MIPS
        |
4. qemu-arm-static -L ./squashfs-root/ ./squashfs-root/usr/sbin/httpd
   -> run the web server locally, fuzz it, find crashes
        |
5. qemu-arm-static -g 1234 ...  + gdb-multiarch
   -> debug the crash, understand the bug, build exploit
        |
6. qemu-system-arm ... (full boot)
   -> test the complete exploit chain end-to-end
        |
7. Test on real hardware to confirm reliability
```

User mode is for finding and debugging bugs fast.
System mode is for testing the full chain before you go on stage.

---

## Common flags reference

### User mode
| Flag | What it does |
|------|-------------|
| -L path | Set library search path (point to firmware's rootfs) |
| -g port | Start GDB server on this port |
| -strace | Print syscalls (like strace) |
| -E VAR=val | Set environment variable |

### System mode
| Flag | What it does |
|------|-------------|
| -M machine | Machine type (virt, malta, versatilepb) |
| -kernel file | Kernel image |
| -drive file=x | Disk image |
| -append "..." | Kernel command line |
| -nographic | Serial console, no GUI |
| -net user,hostfwd=tcp::HOST-:GUEST | Port forwarding |
| -m 256M | RAM size |
| -smp 2 | Number of CPUs |
