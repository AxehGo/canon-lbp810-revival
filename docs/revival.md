# Canon LBP-810 Revival

Technical preservation and documentation of the 2026 revival of the Canon LBP-810 as a network printer using modern Linux infrastructure and the historical CAPT 0.1 driver.

## 1. Project scope

The purpose of the project was to determine whether a Canon LBP-810 could still operate as a network printer in a modern Linux environment.

The working architecture combined:

- a Raspberry Pi Zero 2 W;
- a modern Linux system;
- CUPS;
- Avahi;
- the historical CAPT 0.1 driver;
- IPP clients on Windows and Kubuntu.

The project progressed from initial driver and USB experiments to a working print server, then to a reproducible reference system and a verified full-disk restoration.

## 2. Historical driver

The CAPT driver used by this project was originally written in 2004 by:

**Nicolas Boichat**

Original project:

https://www.boichat.ch/nicolas/capt/index.html

Original release:

`capt-0.1` — version 0.1, August 2004

The original source archive is preserved in this repository as:

`historical/capt-0.1.tar.gz`

The historical driver is not our software. Its original copyright notices, licensing information and historical attributions are preserved in the archive.

The original project also acknowledges code derived from Rildo Pragana's Samsung ML-85G driver.

## 3. Hardware

### Print server

Raspberry Pi Zero 2 W

### Printer

Canon LBP-810

USB identification observed during the experiment:

```text
04a9:260a Canon, Inc. LBP810
```

The Linux printer device used by the working system was:

```text
/dev/usb/lp0
```

The full-disk Gold Master was created from the micro-SD card used by the working server.

## 4. Working Linux environment

The final reference system used:

```text
Armbian 26.8.3 resolute
Ubuntu 26.04
aarch64 / arm64
kernel 6.18.44-current-bcm2711
```

Network management used:

```text
systemd-networkd
wpa_supplicant
```

The system hostname was:

```text
rpizero2w
```

## 5. Printing stack

The validated printing path was:

```text
Windows / Kubuntu
        ↓
       IPP
        ↓
       CUPS
        ↓
    Foomatic
        ↓
      CAPT
        ↓
  /dev/usb/lp0
        ↓
  Canon LBP-810
```

The main components were:

```text
CUPS
Avahi
Foomatic
Ghostscript
CAPT 0.1
Canon LBP-810 PPD
```

CUPS detected the installed PPD as:

```text
Canon-LBP-810-capt.ppd Canon LBP-810 Foomatic/capt (recommended)
```

The final working queue used:

```text
file:///dev/null
```

In this configuration, the CUPS queue remained the logical print destination while the CAPT chain handled communication with the USB printer.

## 6. Building CAPT 0.1

The historical source was compiled on the target Linux system without modifying `capt.c`.

The validated build procedure was:

```bash
make clean
make CFLAGS="-std=gnu89 -O2 -g"
```

The build completed successfully.

Installation was performed with:

```bash
sudo make install
```

The installation provided:

```text
/usr/bin/capt
/usr/bin/capt-print
/usr/share/cups/model/Canon-LBP-810-capt.ppd
```

The use of GNU89 was necessary for the historical source to compile correctly with the modern compiler environment used during the experiment.

## 7. CUPS configuration

The installed PPD was indexed by CUPS and used for the Canon LBP-810 queue.

The validated configuration retained:

```text
Canon LBP-810 Foomatic/capt (recommended)
device-uri=file:///dev/null
```

CUPS accepted jobs and the queue became operational.

The server was then able to provide the printer to network clients through IPP.

## 8. Validation

Validation was performed in stages.

### USB

The printer was detected and the Linux USB printer device was present.

### CUPS

The historical PPD was recognized and the queue became operational.

### Kubuntu

The existing IPP path from Kubuntu continued to work.

### Windows

Windows successfully submitted print jobs through IPP.

### Physical printing

The Canon LBP-810 produced real printed pages.

The final validation therefore covered the complete chain from client submission to physical output.

## 9. Network stability

The earlier Raspberry Pi OS Bookworm installation had shown intermittent Wi-Fi connectivity problems after periods of inactivity.

The experiment was repeated on the same Raspberry Pi Zero 2 W using Armbian.

Several variables changed between the two systems, including architecture, kernel and network stack. Consequently, the experiment does not identify a single component as the cause of the earlier behaviour.

What was observed was:

```text
Same Raspberry Pi Zero 2 W
Same physical environment
Same local network
Different operating system
```

Under Armbian, the server remained reachable for extended periods without a permanent ping and without additional network tweaks.

The conclusion recorded by the experiment is therefore limited to the observed result: Armbian provided better Wi-Fi stability in this specific setup.

## 10. Gold Master

Once the server was fully functional, the Raspberry Pi was shut down cleanly and the micro-SD card was accessed from Kubuntu/Linux.

The full storage device was read with:

```text
USBImager 1.0.10
```

The operation produced:

```text
usbimager-20260915T2315.dd.zst
```

The image used Zstandard compression.

SHA-256:

```text
3df3ba1f96060a645fd93fcd74663e0c4e3f8ccf87096b24d771ab4e7f0b2c35
```

The source card contained:

```text
30 702 592 sectors
512 bytes per sector
```

The Gold Master is intentionally not stored in GitHub. It is a separate system backup.

This repository stores the knowledge, source material, procedures and documentation needed to understand and reproduce the server; the full disk image remains outside Git.

## 11. Restoration

A second micro-SD card was prepared for the restoration experiment.

The Gold Master was written with USBImager with verification enabled.

The restoration completed in:

```text
00:16:27
```

USBImager reported successful completion.

The restored card was placed back in the Raspberry Pi Zero 2 W.

The system booted successfully and retained the expected reference environment:

```text
Armbian 26.8.3
Ubuntu 26.04
kernel 6.18.44-current-bcm2711
CUPS
Avahi
CAPT 0.1
Canon LBP-810 configuration
```

The `/dev/usb/lp0` device was present.

The CUPS queue was present.

A real test print was produced successfully.

The full-disk image was therefore verified as a usable method for reconstructing the working server on another micro-SD card.

## 12. Reproducibility

The experiment established several layers of reproducibility.

### Historical source

The original CAPT 0.1 source archive is preserved unchanged.

### Build procedure

The exact compilation command used for the working driver is documented.

### Print stack

The CUPS, Foomatic, Ghostscript, Avahi and CAPT relationships are documented.

### System reference

The final Armbian environment and kernel version are recorded.

### Gold Master

The complete working system state is preserved separately as a full-disk image.

### Restoration

The Gold Master was restored to another micro-SD card and the resulting system was booted and tested successfully.

The complete chain is therefore:

```text
Historical source
        ↓
Build procedure
        ↓
Print configuration
        ↓
Working server
        ↓
Gold Master
        ↓
Verified restoration
```

## 13. Repository structure

The repository deliberately separates historical material from the 2026 revival work.

```text
canon-lbp810-revival/
│
├── README.md
├── .gitignore
│
├── historical/
│   └── capt-0.1.tar.gz
│
├── docs/
│
├── build/
│
├── config/
│
└── scripts/
```

The historical archive is preserved without modification.

The Gold Master disk image is excluded from Git through `.gitignore`.

The working repository is intended to hold source material, documentation, reproducible procedures and supporting scripts — not a copy of the production machine itself.

## 14. Credits and attribution

### Original CAPT driver

**Nicolas Boichat**

Original project:

https://www.boichat.ch/nicolas/capt/index.html

### Additional historical attribution

The original project acknowledges work derived from:

**Rildo Pragana**

The original attribution remains part of the historical source.

### 2026 revival

Documentation, experimentation, validation and preservation work:

**AxehGo & DebIA**

This is an independent preservation and revival project. It is not affiliated with or endorsed by Canon Inc.

## 15. License

The historical CAPT driver is distributed under the license included with the original source archive.

The original `COPYING` file is preserved inside:

```text
historical/capt-0.1.tar.gz
```

When redistributing or modifying the historical driver, its original copyright notices, license and attribution information must be preserved.

The new documentation and experimental material in this repository are separate from the original CAPT source and should not be confused with the work of Nicolas Boichat.
