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
   
#### Path B: The Universal Alternative — Switching to `.xz` (Recommended)
If you want a compression option that behaves identically to `.imgc` (skipping block zeroes flawlessly) but is natively multi-platform, you can compress your raw file directly on your Chromebook terminal using `xz`:

   ```bash
   xz -vk -9 firmware.img
```
#### 💾 Decompressing the Firmware Image (.img.xz)

The factory-original 3.6 GB full-disk layout has been compressed using `xz` to keep the distribution size compact while preserving raw structure.

#### 🐧 On Linux / Chromebook:
Decompress it instantly using the built-in system utility:
   ```bash
   xz -dk firmware.img.xz

```


#### 🚀 How to Clone and Setup This Repository

Because this repository links directly to the official upstream EDL project, clone this repository along with its submodules by running:

   ```bash
   git clone --recursive [https://github.com/danielw3b/Alcatel-One-Touch-Fire-E-6015X.git](https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git)
   cd Alcatel-One-Touch-Fire-E-6015X

```

#### 💻 Environment Setup (ARM64 Chromebook / Debian Linux)
Standard EDL setups assume an x86_64 architecture. To compile the necessary binary hooks for an ARM64 processor (like those found in many modern Chromebooks), you must install system build tools manually before python packages can compile successfully.

1. Install System Dependencies
Update your Linux container and install the necessary compiler frameworks:

   ```bash
   sudo apt update
   sudo apt install python3-pip python3-venv git build-essential cmake libusb-1.0-0-dev -y
```
2. Create and Activate an Isolated Virtual Environment
   ```bash
   python3 -m venv qdl_env
   source qdl_env/bin/activate

3. Install Python Requirements
Because pre-compiled wheels for keystone-engine do not exist for ARM64 Linux on PyPI, your Chromebook will use cmake to manually build the internal C++ bindings from source during this step. This step will take 5 to 10 minutes to complete. Do not interrupt it.

   ```bash
   pip3 install -r edl/requirements.txt
```
#### 🔌 Hardware Hookup Sequence (Bypassing Boot Loops)
If your device's partition table is completely corrupt, it will dynamically bounce between Fastboot mode and Qualcomm Emergency Download mode, making it difficult for the container to grasp the USB interface. Use this precise physical sequence to latch it reliably:

Disconnect the phone completely from your Chromebook.

Hold Volume Down (-) and the Power Button simultaneously for roughly 8 to 10 seconds until you feel a brief vibration.

Immediately plug the USB cable into the Chromebook the moment the vibration stops.

ChromeOS Prompt: A notification banner will slide out in ChromeOS asking to pass the device to Linux. Click Connect to Linux.

To verify the phone successfully locked into standard QDL mode, open your terminal and run:

   ```bash
   lsusb
   Look for a device listed with a Qualcomm endpoint, usually matching or resembling:

   ID 05c6:9026 Qualcomm Qualcomm CDMA Technologies MSM
```
#### ⚡ Flashing Command
Decompress your downloaded image back into a raw 3.6GB file named firmware.img and place it in your workspace directory.

Because ChromeOS heavily restricts unprivileged user-space access to raw USB communication buses, you must point directly to your virtual environment's Python binary inside a root execution (sudo) wrapper:

   ```bash
   sudo ~/YOUR_REPO_NAME/qdl_env/bin/python3 edl/edl.py wf firmware.img --loader=MSM8610.mbn --memory=emmc
```
Expected Output & Behavior Notes:
Payload Size Warning: You will see an output stating firehose - [LIB]: Host's payload to target size is too large. This is completely normal behavior. The script is acknowledging that the Alcatel 6015X's older RAM buffer size requires smaller data chunks and will automatically downscale the packet transfer window to match.

Progress: The script will map physical partition 0, sector 0, and reconstruct all 7,634,944 blocks of storage space.

Once the terminal prints Wrote firmware.img to sector 0 and returns to your prompt, safely unplug the phone USB cable and hold the Power Button down for 15 seconds to force the Qualcomm processor out of EDL mode and trigger a normal boot up sequence!


#### Example output of installation and flashing:
```bash
#(qdl_env) dan@penguin:~/6015X$ ~/qdl_env/bin/pip3 install -r edl/requirements.txt
Collecting wheel
  Using cached wheel-0.47.0-py3-none-any.whl (32 kB)
Collecting pyusb>=1.1.0
  Using cached pyusb-1.3.1-py3-none-any.whl (58 kB)
Requirement already satisfied: pyserial>=3.4 in /home/dan/qdl_env/lib/python3.11/site-packages (from -r edl/requirements.txt (line 3)) (3.5)
Collecting docopt>=0.6.2
  Using cached docopt-0.6.2.tar.gz (25 kB)
  Preparing metadata (setup.py) ... done
Collecting pycryptodome
  Using cached pycryptodome-3.23.0-cp37-abi3-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (2.2 MB)
Collecting pycryptodomex
  Using cached pycryptodomex-3.23.0-cp37-abi3-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (2.2 MB)
Collecting lxml>=4.6.1
  Using cached lxml-6.1.1-cp311-cp311-manylinux_2_26_aarch64.manylinux_2_28_aarch64.whl (5.0 MB)
Collecting colorama
  Using cached colorama-0.4.6-py2.py3-none-any.whl (25 kB)
Collecting capstone
  Using cached capstone-5.0.9-py3-none-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (1.5 MB)
Collecting keystone-engine
  Using cached keystone-engine-0.9.2.tar.gz (2.8 MB)
  Preparing metadata (setup.py) ... done
Collecting qrcode
  Using cached qrcode-8.2-py3-none-any.whl (45 kB)
Collecting requests
  Using cached requests-2.34.2-py3-none-any.whl (73 kB)
Requirement already satisfied: passlib in /home/dan/qdl_env/lib/python3.11/site-packages (from -r edl/requirements.txt (line 13)) (1.7.4)
Collecting Exscript
  Using cached exscript-2.6.32-py2.py3-none-any.whl (255 kB)
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProtocolError('Connection aborted.', RemoteDisconnected('Remote end closed connection without response'))': /simple/packaging/
Collecting packaging>=24.0
  Using cached packaging-26.2-py3-none-any.whl (100 kB)
Collecting charset_normalizer<4,>=2
  Using cached charset_normalizer-3.4.7-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl (206 kB)
Collecting idna<4,>=2.5
  Using cached idna-3.18-py3-none-any.whl (65 kB)
Collecting urllib3<3,>=1.26
  Using cached urllib3-2.7.0-py3-none-any.whl (131 kB)
Collecting certifi>=2023.5.7
  Using cached certifi-2026.6.17-py3-none-any.whl (133 kB)
Collecting future
  Using cached future-1.0.0-py3-none-any.whl (491 kB)
Collecting configparser
  Using cached configparser-7.2.0-py3-none-any.whl (17 kB)
Collecting paramiko<5,>=1.17
  Using cached paramiko-4.0.0-py3-none-any.whl (223 kB)
Collecting bcrypt>=3.2
  Using cached bcrypt-5.0.0-cp39-abi3-manylinux_2_34_aarch64.whl (275 kB)
Collecting cryptography>=3.3
  Using cached cryptography-49.0.0-cp311-abi3-manylinux_2_34_aarch64.whl (4.7 MB)
Collecting invoke>=2.0
  Using cached invoke-3.0.3-py3-none-any.whl (160 kB)
Collecting pynacl>=1.5
  Using cached pynacl-1.6.2-cp38-abi3-manylinux_2_34_aarch64.whl (818 kB)
Collecting cffi>=2.0.0
  Using cached cffi-2.0.0-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.whl (216 kB)
Collecting pycparser
  Using cached pycparser-3.0-py3-none-any.whl (48 kB)
Installing collected packages: keystone-engine, docopt, urllib3, qrcode, pyusb, pycryptodomex, pycryptodome, pycparser, packaging, lxml, invoke, idna, future, configparser, colorama, charset_normalizer, certifi, capstone, bcrypt, wheel, requests, cffi, pynacl, cryptography, paramiko, Exscript
  DEPRECATION: keystone-engine is being installed using the legacy 'setup.py install' method, because it does not have a 'pyproject.toml' and the 'wheel' package is not installed. pip 23.1 will enforce this behaviour change. A possible replacement is to enable the '--use-pep517' option. Discussion can be found at https://github.com/pypa/pip/issues/8559
  Running setup.py install for keystone-engine ... done
  DEPRECATION: docopt is being installed using the legacy 'setup.py install' method, because it does not have a 'pyproject.toml' and the 'wheel' package is not installed. pip 23.1 will enforce this behaviour change. A possible replacement is to enable the '--use-pep517' option. Discussion can be found at https://github.com/pypa/pip/issues/8559
  Running setup.py install for docopt ... done
Successfully installed Exscript-2.6.32 bcrypt-5.0.0 capstone-5.0.9 certifi-2026.6.17 cffi-2.0.0 charset_normalizer-3.4.7 colorama-0.4.6 configparser-7.2.0 cryptography-49.0.0 docopt-0.6.2 future-1.0.0 idna-3.18 invoke-3.0.3 keystone-engine-0.9.2 lxml-6.1.1 packaging-26.2 paramiko-4.0.0 pycparser-3.0 pycryptodome-3.23.0 pycryptodomex-3.23.0 pynacl-1.6.2 pyusb-1.3.1 qrcode-8.2 requests-2.34.2 urllib3-2.7.0 wheel-0.47.0

#(qdl_env) dan@penguin:~/6015X$ python3 edl/edl.py wf firmware.img --loader=MSM8610.mbn --memory=emmc
Qualcomm Sahara / Firehose Client V3.62 (c) B.Kerler 2018-2025.
main - Using loader MSM8610.mbn ...
main - Waiting for the device
main - Device detected :)
sahara - Protocol version: 2, Version supported: 1
main - Mode detected: sahara
sahara - 
Version 0x2
------------------------
HWID:              0x008110e100000000 (MSM_ID:0x008110e1,OEM_ID:0x0000,MODEL_ID:0x0000)
CPU detected:      "MSM8210"
PK_HASH:           0xcc3153a80293939b90d02d3bf8b23e0292e452fef662c74998421adad42a380f
Serial:            0x0472d430

sahara - Protocol version: 2, Version supported: 1
sahara - Uploading loader MSM8610.mbn ...
sahara - 32-Bit mode detected.
sahara - Loader successfully uploaded.
main - Trying to connect to firehose loader ...
firehose
firehose - [LIB]: Host's payload to target size is too large
firehose
firehose - [LIB]: !DEBUG! rsp.data: 'bytearray(b'<?xml version="1.0" encoding="UTF-8" ?><data><log value="logbuf@0xF8021118 fh@0xF801DF80" /></data>')'
firehose - TargetName=MSM8x10
firehose - MemoryName=eMMC
firehose - Version=1
firehose - Trying to read first storage sector...
firehose - Running configure...
firehose_client - Supported functions:
-----------------
firehose - 
Writing to physical partition 0, sector 0, sectors 7634944
Progress: |██████████| 100.0% Write (Sector 0x747120 of 0x748000, ) 2.56 MB/s                            
Progress: |██████████| 100.0% Write (Sector 0x747180 of 0x748000, ) 2.89 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7471E0 of 0x748000, ) 5.04 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747240 of 0x748000, ) 5.33 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7472A0 of 0x748000, ) 5.59 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747300 of 0x748000, ) 5.40 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747360 of 0x748000, ) 6.21 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7473C0 of 0x748000, ) 4.80 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747420 of 0x748000, ) 6.01 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747480 of 0x748000, ) 4.21 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7474E0 of 0x748000, ) 4.58 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747540 of 0x748000, ) 5.61 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7475A0 of 0x748000, ) 5.72 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747600 of 0x748000, ) 5.83 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747660 of 0x748000, ) 5.67 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7476C0 of 0x748000, ) 6.19 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747720 of 0x748000, ) 6.22 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747780 of 0x748000, ) 7.01 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7477E0 of 0x748000, ) 7.72 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747840 of 0x748000, ) 5.39 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7478A0 of 0x748000, ) 6.59 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747900 of 0x748000, ) 5.29 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747960 of 0x748000, ) 6.19 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x7479C0 of 0x748000, ) 4.62 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747A20 of 0x748000, ) 6.79 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747A80 of 0x748000, ) 5.75 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747AE0 of 0x748000, ) 6.44 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747B40 of 0x748000, ) 5.87 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747BA0 of 0x748000, ) 6.44 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747C00 of 0x748000, ) 4.97 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747C60 of 0x748000, ) 5.91 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747CC0 of 0x748000, ) 5.43 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747D20 of 0x748000, ) 5.60 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747D80 of 0x748000, ) 4.96 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747DE0 of 0x748000, ) 5.27 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747E40 of 0x748000, ) 5.82 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747EA0 of 0x748000, ) 5.53 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747F00 of 0x748000, ) 5.31 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747F60 of 0x748000, ) 6.64 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x747FC0 of 0x748000, ) 5.81 MB/s                          
Progress: |██████████| 100.0% Write (Sector 0x748000 of 0x748000, ) 5.52 MB/s                          
Wrote firmware.img to sector 0.
```

### 🤖 AI-Assisted Development
This unbricking methodology and documentation were co-developed with Gemini, leveraging automated partition analysis to tailor a Qualcomm Emergency Download (EDL) recovery workflow specifically optimized for ARM64 Chromebook Linux (Crostini) environments. By cross-referencing raw sector maps with open-source flashing tools, this approach establishes a reproducible pipeline for restoring legacy mobile hardware without requiring native Windows systems.


### ⚠️ Forcing QDL Mode From an Active Fastboot Loop

If your device is currently accessible via Fastboot (`fastboot devices`) but refuses to lock into Qualcomm Diagnostics/QDL mode, you can manually force a hardware fallback by erasing the secondary bootloaders. This strips away the broken software boot chain and forces the phone's primary internal ROM to drop straight into QDL mode upon restart.

Run the following commands in sequence while the phone is in Fastboot mode:

```bash
fastboot erase boot
fastboot erase aboot
fastboot erase sbl1
fastboot reboot
```

When a Qualcomm device is stuck bouncing between states, intentionally destroying the corrupted or incompatible software bootloaders via Fastboot is the ultimate way to trigger a "hard fallback." By erasing sbl1 (Secondary Bootloader) and aboot (Applications Bootloader/Fastboot itself), the phone has absolutely no instructions to execute upon restarting. The hard-coded Primary Bootloader (PBL) inside the CPU detects this total vacuum and immediately forces the phone into QDL (Emergency Download 9008) mode.
