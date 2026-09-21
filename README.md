# Canon LBP-810 Revival

Documentation and preservation of a working Linux print-server configuration for the Canon LBP-810, using the historical CAPT 0.1 driver.

## Original CAPT driver

The driver preserved in this repository is **not our software**.

It was originally written in 2004 by:

**Nicolas Boichat**

Original project:
https://www.boichat.ch/nicolas/capt/index.html

Original release:
`capt-0.1` — version 0.1, August 2004

The original source identifies Nicolas Boichat as the copyright holder and is released under the **GNU General Public License, version 2 (GPL-2.0)**. The original source also acknowledges code derived from Rildo Pragana's Samsung ML-85G driver.

The original copyright and license notices are preserved.

## What this repository contains

This repository separates the historical driver from the 2026 revival work:

- the original CAPT 0.1 source and PPD;
- build and installation notes;
- CUPS configuration used during the revival;
- Raspberry Pi / Armbian configuration notes;
- test procedures and documented results;
- scripts and supporting documentation created for the revival.

## The 2026 revival

The goal was to determine whether a Canon LBP-810, released in the early 2000s, could still operate as a network printer in a modern Linux environment.

The working system was rebuilt around a Raspberry Pi Zero 2 W with CUPS, Avahi, CAPT 0.1 and IPP clients.

The project also produced a reproducible reference system (the Gold Master) and a verified full-disk restoration.

The modern documentation and experimental work in this repository are by **AxehGo & DebIA**.

## Attribution

Please preserve the original authorship, copyright and license information when redistributing or modifying the historical driver.

This repository is an independent preservation and revival project. It is not affiliated with or endorsed by Canon Inc.
