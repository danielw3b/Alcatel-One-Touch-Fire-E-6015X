# Alcatel One Touch Fire E (6015X) Full EDL Unbricking Guide

This repository contains the specific methodology, hardware sequences, and target loader required to recover a hard-bricked Alcatel One Touch Fire E (6015X) Firefox OS device from a completely wiped GUID Partition Table (GPT) boot-loop.

Crucially, this document provides the exact workaround instructions required to successfully compile dependencies and execute this flash from an **ARM64 Chromebook Linux (Crostini) environment**.

## 🛠️ Repository Components & Dependencies
* **Target Loader:** `MSM8610.mbn` (Included in the root of this repo).
* **Upstream EDL Tool:** Powered by [Bkerler's EDL Tool](https://github.com/bkerler/edl).
* **Full Disk Factory Image (~820MB Compressed / 3.6GB Raw `.img`):
* **[👉 [https://drive.google.com/file/d/1Sk7TzT1cnt5ahuoYGpkpCab_rqu2HTB6/view?usp=drive_link] 👈]

### 💾 Restoring the Compressed Firmware Image (.imgc)

The factory-original dump is provided as a compressed raw format (`.imgc`) to minimize storage footprint.
#### Path A:
#### 🪟 On Windows:
1. Download and launch the portable tool from [HDD Guru Raw Copy Tool](https://hddguru.com/software/HDD-Raw-Copy-Tool/).
2. Double-click the **FILE** option as your source, and select `firmware.imgc`.
3. Click **Continue**, select your destination (a raw file or raw drive target), and click **Start**.

#### 🐧 On Linux / Chromebook (Crostini):
Because `.imgc` is a proprietary container format, you must compile an unpacker to decompress it natively under Linux:
1. Clone and build the `unimgc` utility:
   ```bash
   git clone [https://github.com/shizmob/unimgc.git](https://github.com/shizmob/unimgc.git)
   cd unimgc
   make
   sudo cp unimgc /usr/local/bin/

   unimgc -v firmware.imgc firmware.img

---
#### Path B: The Universal Alternative — Switching to `.xz` (Recommended)
If you want a compression option that behaves identically to `.imgc` (skipping block zeroes flawlessly) but is natively multi-platform, you can compress your raw file directly on your Chromebook terminal using `xz`:

   ```bash
   xz -vk -9 firmware.img

---

