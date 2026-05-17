# Pwn2Own Tooling Setup

## Current status (as of 2026-05-16)

### Have (ready)
- qemu-system-arm, qemu-system-mipsel, qemu-arm-static, qemu-mipsel-static
- Caido, nmap, ffuf, nuclei
- Python 3.10, Node 22, Go 1.25, Ruby, Rust
- strace, ltrace, objdump, readelf, strings, file
- Docker, unsquashfs
- 12 cores, 23GB RAM

### Missing (must install)

```bash
# 1. Reverse engineering - Ghidra (free, essential)
sudo apt install -y default-jdk
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.3.2_build/ghidra_11.3.2_PUBLIC_20250415.zip
unzip ghidra_11.3.2_PUBLIC_20250415.zip -d ~/tools/

# 2. GDB + pwndbg (debugger + exploit dev plugin)
sudo apt install -y gdb gdb-multiarch
git clone https://github.com/pwndbg/pwndbg ~/tools/pwndbg
cd ~/tools/pwndbg && ./setup.sh

# 3. pwntools (exploit scripting framework)
pip3 install pwntools

# 4. Binwalk (firmware extraction)
pip3 install binwalk

# 5. ROPgadget (ROP chain building)
pip3 install ROPgadget

# 6. checksec (check binary protections)
pip3 install checksec.py

# 7. Cross-compilers (build payloads for ARM/MIPS targets)
sudo apt install -y gcc-arm-linux-gnueabi gcc-mipsel-linux-gnu

# 8. Wireshark/tshark (network protocol analysis)
sudo apt install -y wireshark tshark

# 9. One-gadget (find magic gadgets in libc)
sudo gem install one_gadget

# 10. Ropper (another ROP gadget finder)
pip3 install ropper
```

### Nice-to-have

```bash
# Firmware emulation framework
pip3 install firmwalker
git clone https://github.com/attify/firmware-analysis-toolkit ~/tools/fat

# Sasquatch (handles non-standard SquashFS from vendors)
git clone https://github.com/devttys0/sasquatch ~/tools/sasquatch
cd ~/tools/sasquatch && ./build.sh

# Hashcat (password cracking, useful for extracted firmware creds)
sudo apt install -y hashcat
```

## Disk warning

At 90% usage (45GB free). Some tools + firmware images need space.
Consider freeing up or adding storage before heavy firmware work.
