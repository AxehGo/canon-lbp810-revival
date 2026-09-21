# Canon LBP-810 — Rebuild procedure

This document describes the reconstruction procedure validated during the Canon LBP-810 Revival experiment.

It is deliberately separate from the historical driver source. The original CAPT 0.1 archive is preserved under `historical/` and remains attributed to Nicolas Boichat.

## 1. Scope

The procedure targets the final validated environment used by the project:

- Raspberry Pi Zero 2 W
- Armbian 26.8.3 resolute
- Ubuntu 26.04
- aarch64 / arm64
- kernel 6.18.44-current-bcm2711
- CUPS
- Avahi
- Ghostscript
- Foomatic / CUPS filters
- CAPT 0.1
- Canon LBP-810

The procedure describes the working configuration documented by the 2026 experiment. It is not a guarantee that future Linux releases will behave identically.

## 2. Install and update the base system

After installing Armbian:

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

After reboot, verify the running kernel:

```bash
uname -r
```

The final validated system used:

```text
6.18.44-current-bcm2711
```

Do not assume that an installed kernel is the kernel currently running. During the experiment, an installed 6.18.44 kernel was present while the system was still running 6.18.42. The USB printer module belonged to 6.18.44, so `modprobe` could not find it until the system was rebooted into the matching kernel.

## 3. Basic network validation

The final server used:

```text
Hostname: rpizero2w
IPv4:    192.168.1.40/24
```

The active network stack was based on:

```text
systemd-networkd
wpa_supplicant
```

Basic checks:

```bash
hostname
ip -br addr
ip route
systemctl is-active systemd-networkd
systemctl is-active wpa_supplicant
```

The exact local IP address is environment-specific. Do not copy the example address blindly into another network.

## 4. Install the print-server software

The validated package installation was:

```bash
sudo apt install cups cups-client cups-filters ghostscript avahi-daemon build-essential
```

Then configure CUPS administration and network access:

```bash
sudo usermod -aG lpadmin admin
sudo cupsctl --remote-any
sudo systemctl restart cups
```

Enable the services:

```bash
sudo systemctl enable --now cups
sudo systemctl enable --now avahi-daemon
```

Verify:

```bash
systemctl is-active cups
systemctl is-active avahi-daemon
lpstat -t
```

A new CUPS installation may correctly report that no printer exists yet. That is not an error.

## 5. Obtain the historical CAPT source

The original CAPT 0.1 archive is preserved in this repository:

```text
historical/capt-0.1.tar.gz
```

Extract it:

```bash
cd ~
tar xzf ~/canon-lbp810-revival/historical/capt-0.1.tar.gz
cd ~/capt-0.1
```

The archive contains the original source, build files, PPD and historical documentation.

Do not modify the original source merely to make it easier to maintain. The revival experiment demonstrated that the original `capt.c` could be compiled successfully with the appropriate compiler mode.

## 6. Build CAPT 0.1

Compile on the target Raspberry Pi:

```bash
cd ~/capt-0.1
make clean
make CFLAGS="-std=gnu89 -O2 -g"
```

The use of GNU89 was required for the historical source to compile correctly with the modern compiler environment used during the experiment.

Install:

```bash
sudo make install
```

Verify the installed files:

```bash
ls -l /usr/bin/capt
ls -l /usr/bin/capt-print
ls -l /usr/share/cups/model/Canon-LBP-810-capt.ppd
```

## 7. Verify the USB printer path

With the Canon connected:

```bash
lsusb
```

The validated USB identification was:

```text
04a9:260a Canon, Inc. LBP810
```

The CAPT program communicates through the Linux USB printer device.

First check the kernel configuration:

```bash
grep -E 'CONFIG_USB_PRINTER|CONFIG_USB_USBLP' /boot/config-$(uname -r)
```

The validated kernel reported:

```text
CONFIG_USB_PRINTER=m
```

Then:

```bash
modinfo usblp
sudo modprobe usblp
ls -l /dev/usb/lp0
```

The expected device was:

```text
/dev/usb/lp0
```

During the experiment, a missing `usblp` error was traced to a kernel-version mismatch: the system was still running 6.18.42 while the `usblp.ko` module belonged to 6.18.44. The fix was to reboot into the matching kernel, not to rebuild CUPS or the kernel.

## 8. Verify the CAPT PPD

Check that CUPS sees the historical PPD:

```bash
lpinfo -m | grep -i 'LBP-810'
```

The validated result was:

```text
Canon-LBP-810-capt.ppd Canon LBP-810 Foomatic/capt (recommended)
```

## 9. Create the CUPS queue

The final working configuration deliberately used `file:///dev/null` as the CUPS device URI.

Create the queue:

```bash
sudo lpadmin -p LBP810 -E \
  -v file:///dev/null \
  -m Canon-LBP-810-capt.ppd \
  -o printer-is-shared=true
```

Enable and accept jobs:

```bash
sudo cupsenable LBP810
sudo cupsaccept LBP810
```

Verify:

```bash
lpstat -p LBP810
lpstat -v LBP810
lpoptions -p LBP810
```

The important properties are:

```text
printer LBP810 is idle. enabled
device-uri=file:///dev/null
printer-is-shared=true
printer-make-and-model='Canon LBP-810 Foomatic/capt (recommended)'
```

The `file:///dev/null` URI is intentional in this architecture. The CAPT chain, rather than the CUPS USB backend, is responsible for the direct communication with `/dev/usb/lp0`.

## 10. Why Ghostscript is required

The historical `capt-print` script uses Ghostscript as the rasterizer.

The effective pipeline is:

```text
PostScript
    ↓
Ghostscript
    ↓
PBM raw
    ↓
capt
    ↓
/dev/usb/lp0
    ↓
Canon LBP-810
```

Therefore Ghostscript is not an optional extra for this historical driver configuration.

## 11. First real print test

Use the standard CUPS test document:

```bash
lp -d LBP810 /usr/share/cups/data/testprint
```

Then verify the queue:

```bash
lpstat -p LBP810
```

The project treated a successful physical page as the decisive validation of the local print chain.

## 12. Network printing

Once local printing works, test the IPP path from a client computer.

The project validated both:

- Windows over IPP;
- Kubuntu over IPP.

Avahi was used for service discovery on the local network.

The exact client-side setup can vary by operating system and should not be confused with the Raspberry Pi's local CUPS configuration.

## 13. Gold Master

The Gold Master is deliberately outside Git.

After the server reached a known-good state, the Raspberry Pi was shut down cleanly and the complete micro-SD device was imaged from Kubuntu/Linux using USBImager 1.0.10.

Reference image:

```text
usbimager-20260915T2315.dd.zst
```

Compression:

```text
Zstandard
```

SHA-256:

```text
3df3ba1f96060a645fd93fcd74663e0c4e3f8ccf87096b24d771ab4e7f0b2c35
```

The source device contained:

```text
30 702 592 sectors
512 bytes per sector
```

The image is a full-disk image, not a collection of individual partition backups.

It must not be stored in this Git repository.

## 14. Gold Master restoration

For a restoration test, write the full image to a compatible micro-SD card using USBImager.

The project used verification during the write.

The validated restoration completed in:

```text
00:16:27
```

The restored card booted successfully in the Raspberry Pi Zero 2 W.

Validation after restoration included:

```bash
uname -r
systemctl --failed
systemctl is-active cups
systemctl is-active avahi-daemon
lpstat -r
lpstat -p -d
lsusb
ls -l /dev/usb/lp0
lpoptions -p LBP810
```

A physical print was then produced successfully.

## 15. What belongs in Git and what does not

GitHub contains the project knowledge and reconstruction material:

- historical CAPT source archive;
- documentation;
- build procedure;
- CUPS configuration notes;
- reproducibility notes;
- supporting scripts.

The Gold Master and machine-specific secrets remain outside GitHub.

Examples of material that must not be committed:

```text
*.img
*.dd
*.dd.zst
*.raw
*.raw.zst
.env
*.key
*.pem
private local configuration
```

The repository `.gitignore` already excludes the disk-image patterns used by the project.

## 16. Historical attribution

The CAPT driver is historical software written by:

**Nicolas Boichat**

Original project:

https://www.boichat.ch/nicolas/capt/index.html

The historical source also contains its original attribution to work derived from Rildo Pragana's Samsung ML-85G driver.

The original copyright and license files are preserved in the archive.

The 2026 documentation and revival work are by:

**AxehGo & DebIA**

This project is independent of Canon Inc.

## 17. Reproducibility boundary

The following parts were demonstrated as reproducible:

- compilation of CAPT 0.1 on the target Linux system;
- installation of the historical CAPT files;
- detection of the Canon LBP-810;
- creation of the CUPS queue;
- physical local printing;
- IPP printing from Windows and Kubuntu;
- capture of the complete reference system;
- restoration of the reference system to another micro-SD card.

The precise behaviour of future Linux distributions, kernels, CUPS releases, compilers and hardware remains outside the scope of this validated experiment.
