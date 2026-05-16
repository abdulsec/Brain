# Pwn2Own Learning Roadmap

## What is Pwn2Own

- You register a target + category before the contest (e.g., "SOHO router - WAN to LAN RCE")
- You get 3 attempts, 5 minutes each to demo a full exploit chain live
- Bugs must be 0-days - unpatched at time of contest
- Payout depends on category: $5K-$100K+ per chain

## Which Pwn2Own to target

| Contest | Targets | When |
|---------|---------|------|
| Pwn2Own Austin (SOHO) | Routers, NAS, cameras, printers, smart speakers | ~Oct/Nov |
| Pwn2Own Vancouver | Browsers, VMs, OS privilege escalation, enterprise apps | ~Mar |
| Pwn2Own Ireland (Mobile) | Phones, messaging apps | ~Oct |
| Pwn2Own Automotive | IVI, EV chargers, Tesla | ~Jan |

## Vuln classes that win Pwn2Own

- Memory corruption (heap overflow, UAF, type confusion) - C/C++ targets
- Auth bypass -> RCE chains
- Command injection in web interfaces - very common in SOHO routers
- Stack overflow in network daemons (UPnP, SSDP, DNS)

---

## Phase 1 - Learn how binaries work (2-3 weeks)

Before exploiting anything, you need to understand what a compiled program looks like.

**Start here:**
- **pwn.college** (https://pwn.college) - free ASU course by Zardus (top CTF player). Start from "Program Interaction" module. Walks you through assembly, memory layout, shellcode, everything with hands-on challenges
- **Nightmare** (https://guyinatuxedo.github.io) - real-world binary exploitation tutorial. Start from Chapter 1 "Intro to Assembly"

**What you'll learn:**
- x86 assembly basics (registers, stack, calling conventions)
- How C compiles to machine code
- What the stack/heap look like in memory
- How gdb works

---

## Phase 2 - Learn exploitation basics (3-4 weeks)

**Follow in order:**
1. Buffer overflows - pwn.college "Memory Errors" module
2. ROP chains - pwn.college "Return Oriented Programming" module
3. Heap exploitation - pwn.college "Dynamic Allocator Misuse" module
4. Format strings - pwn.college "Format String Exploits"

**Practice:**
- picoCTF (https://picoctf.org) - beginner CTF, has a "Binary Exploitation" category
- OverTheWire: Narnia (https://overthewire.org/wargames/narnia/) - simple buffer overflow war game
- ROP Emporium (https://ropemporium.com) - dedicated ROP challenges, very well structured

---

## Phase 3 - Learn firmware hacking (2-3 weeks)

This is where it gets specific to Pwn2Own SOHO targets.

**Free courses/resources:**
1. Attify - Firmware Security 101 (https://www.attify.com/iot-security-training) - free intro videos on firmware extraction
2. Wrong Baud's blog (https://wrongbaud.github.io) - practical firmware RE walkthroughs
3. Azeria Labs (https://azeria-labs.com) - best free ARM exploitation tutorials on the internet. Start with:
   - "ARM Assembly Basics"
   - "ARM Exploit Development"
   - "Writing ARM Shellcode"

**Hands-on practice:**
1. Download a real firmware (e.g., old TP-Link or D-Link from their support page)
2. binwalk -e firmware.bin
3. Explore the filesystem - find httpd, cgi-bin scripts, config files
4. Run the web server binary with qemu-arm-static
5. Fuzz it, read the code in Ghidra

**YouTube channels:**
- LiveOverflow - "Binary Exploitation" playlist (start here if you prefer video)
- John Hammond - CTF walkthroughs including binary/firmware
- Ghidra Ninja - Ghidra reverse engineering tutorials

---

## Phase 4 - Study past Pwn2Own winners (ongoing)

Read how people actually won:

- Synacktiv blog - they win Pwn2Own repeatedly, publish detailed writeups
- Team Viettel blog - same, very detailed
- ZDI blog (https://zerodayinitiative.com/blog) - "Pwn2Own" tagged posts explain each winning entry
- Search YouTube: "Pwn2Own router exploit walkthrough"

Key talks to watch:
- "Hacking the Netgear Nighthawk" - multiple DEF CON talks on SOHO router exploitation
- "Pwn2Own Tokyo/Austin" post-event ZDI presentations

---

## Phase 5 - CTFs to sharpen skills (ongoing)

| CTF | Focus | Difficulty |
|-----|-------|-----------|
| picoCTF (https://picoctf.org) | Mixed, good binary intro | Beginner |
| pwn.college (https://pwn.college) | Binary exploitation | Beginner -> Advanced |
| ROP Emporium (https://ropemporium.com) | ROP chains specifically | Intermediate |
| DVRF (https://github.com/praetorian-inc/DVRF) | Router firmware vulns | Intermediate |
| CTFtime (https://ctftime.org) | Live competitions every weekend | All levels |

---

## Suggested daily schedule

| Time | Activity |
|------|----------|
| 1 hour | pwn.college or Azeria Labs tutorial |
| 1 hour | Practice challenge (picoCTF / ROP Emporium) |
| 30 min | Read one Pwn2Own writeup or ZDI blog post |

---

## Honest timeline

| Milestone | Timeframe |
|-----------|-----------|
| Comfortable with GDB + assembly | 1 month |
| Can solve basic CTF pwn challenges | 2 months |
| Can extract + emulate firmware | 3 months |
| Can find real bugs in router firmware | 4-6 months |
| Ready to attempt Pwn2Own entry | 6-12 months |
