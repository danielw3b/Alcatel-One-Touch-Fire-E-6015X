# Alcatel One Touch Fire E (6015X) Full EDL Unbricking Guide

This repository contains the specific methodology, hardware sequences, and target loader required to recover a hard-bricked Alcatel One Touch Fire E (6015X) Firefox OS device from a completely wiped GUID Partition Table (GPT) boot-loop.

Crucially, this document provides the exact workaround instructions required to successfully compile dependencies and execute this flash from an **ARM64 Chromebook Linux (Crostini) environment**.

## 🛠️ Repository Components & Dependencies
* **Target Loader:** `MSM8610.mbn` (Included in the root of this repo).
* **Upstream EDL Tool:** Powered by [Bkerler's EDL Tool](https://github.com/bkerler/edl).
* **Full Disk Factory Image (~820MB Compressed / 3.6GB Raw `.img`):
* **[👉 [https://drive.google.com/file/d/1Sk7TzT1cnt5ahuoYGpkpCab_rqu2HTB6/view?usp=drive_link] 👈]

---

## 🚀 How to Clone and Setup This Repository

Because this repository links directly to the official upstream EDL project, clone this repository along with its submodules by running:

```bash
git clone --recursive [https://github.com/danielw3b/Alcatel-One-Touch-Fire-E-6015X.git](https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git)
cd Alcatel-One-Touch-Fire-E-6015X

💻 Environment Setup (ARM64 Chromebook / Debian Linux)
Standard EDL setups assume an x86_64 architecture. To compile the necessary binary hooks for an ARM64 processor (like those found in many modern Chromebooks), you must install system build tools manually before python packages can compile successfully.

1. Install System Dependencies
Update your Linux container and install the necessary compiler frameworks:

Bash
sudo apt update
sudo apt install python3-pip python3-venv git build-essential cmake libusb-1.0-0-dev -y
2. Create and Activate an Isolated Virtual Environment
Bash
python3 -m venv qdl_env
source qdl_env/bin/activate
3. Install Python Requirements
Because pre-compiled wheels for keystone-engine do not exist for ARM64 Linux on PyPI, your Chromebook will use cmake to manually build the internal C++ bindings from source during this step. This step will take 5 to 10 minutes to complete. Do not interrupt it.

Bash
pip3 install -r edl/requirements.txt
🔌 Hardware Hookup Sequence (Bypassing Boot Loops)
If your device's partition table is completely corrupt, it will dynamically bounce between Fastboot mode and Qualcomm Emergency Download mode, making it difficult for the container to grasp the USB interface. Use this precise physical sequence to latch it reliably:

Disconnect the phone completely from your Chromebook.

Hold Volume Down (-) and the Power Button simultaneously for roughly 8 to 10 seconds until you feel a brief vibration.

Immediately plug the USB cable into the Chromebook the moment the vibration stops.

ChromeOS Prompt: A notification banner will slide out in ChromeOS asking to pass the device to Linux. Click Connect to Linux.

To verify the phone successfully locked into standard QDL mode, open your terminal and run:

Bash
lsusb
Look for a device listed with a Qualcomm endpoint, usually matching or resembling:

ID 05c6:9026 Qualcomm Qualcomm CDMA Technologies MSM

⚡ Flashing Command
Decompress your downloaded image back into a raw 3.6GB file named firmware.img and place it in your workspace directory.

Because ChromeOS heavily restricts unprivileged user-space access to raw USB communication buses, you must point directly to your virtual environment's Python binary inside a root execution (sudo) wrapper:

Bash
sudo ~/YOUR_REPO_NAME/qdl_env/bin/python3 edl/edl.py wf firmware.img --loader=MSM8610.mbn --memory=emmc
Expected Output & Behavior Notes:
Payload Size Warning: You will see an output stating firehose - [LIB]: Host's payload to target size is too large. This is completely normal behavior. The script is acknowledging that the Alcatel 6015X's older RAM buffer size requires smaller data chunks and will automatically downscale the packet transfer window to match.

Progress: The script will map physical partition 0, sector 0, and reconstruct all 7,634,944 blocks of storage space.

Once the terminal prints Wrote firmware.img to sector 0 and returns to your prompt, safely unplug the phone USB cable and hold the Power Button down for 15 seconds to force the Qualcomm processor out of EDL mode and trigger a normal boot up sequence!
