# Intel Ghost Environment (v4.0)

## Overview
The Intel Ghost Environment is a powerful, environment-level daemon designed to bridge the gap between CUDA-centric AI software and Intel Arc/Core Ultra hardware. By injecting a targeted set of oneAPI/SYCL environment variables and native XPU optimizations at runtime, this tool allows Intel GPUs to report their identity as high-end NVIDIA GeForce RTX equivalents and bypass artificial Windows memory limits.

Version 4.0 Massive Update: Ghost is no longer just an environment wrapper. It is now a fully automated Virtual Environment Daemon featuring **Smart Crash Interception** and **Automated Source Translation**. Because Intel does not have a runtime CUDA translator (like AMD's ZLUDA), Ghost introduces the `ghost-translate` command, which leverages Intel SYCLomatic to automatically rewrite CUDA source code into Intel SYCL code.

ROADMAP:

Completed: Native IPEX Injection, Core Ultra VRAM Bypass, Dual-GPU Support, Automated SYCLomatic Translation, Interactive TUI.

Next Big Thing: Full Linux Native Support
Goal: Bring the entire Intel Ghost Environment natively to Ubuntu/Debian.
Status: Windows is fully supported and production-ready. The Linux port is currently in active beta and coming very soon!

Note: Development is currently balanced alongside full-time studies, so updates may be paced accordingly to ensure stability over speed.

## System Requirements & Prerequisites

**Windows:**
* **OS:** Windows 10 or Windows 11.
* **Drivers:** Latest Intel Arc & Iris Xe Graphics Drivers (Wildcat Lake supported).
* **Intel oneAPI Base Toolkit:** *Crucial* for the `ghost-translate` tool and SYCL runtime libraries.
* **Visual Studio Build Tools:** Required for the `cl.exe` C++ compiler if you intend to compile translated SYCL code.
* **PowerShell:** Version 5.1 or newer (Built into Windows).

## Hardware Support Matrix
The internal detection engine provides dynamic spoofing and optimization logic across multiple generations of Intel architectures:

| Intel Host Hardware | Backend Target | NVIDIA Spoof Target |
|:--- |:--- |:--- |
| **Arc A-Series / B-Series (dGPU)** | Level Zero (Discrete) | NVIDIA GeForce RTX 3080 |
| **Core Ultra (iGPU)** | Level Zero (Integrated) | NVIDIA GeForce RTX 3060 |
| **Iris Xe (iGPU)** | Level Zero (Integrated) | NVIDIA GeForce GTX 1650 |

*(Note: Core Ultra and Iris Xe chips share system RAM. Ghost automatically injects allocation bypasses to unlock up to 16GB of dedicated VRAM for these chips, depending on your total system memory).*

## Key Features
* **Native XPU Injection:** Ghost automatically forces PyTorch to utilize the `level_zero` SYCL backend, prioritizing dedicated Arc GPUs over integrated graphics in Dual-GPU setups.
* **Automated CUDA Translator (`ghost-translate`):** Point Ghost at any folder containing NVIDIA CUDA (`.cu`) source code, and it will automatically download SYCLomatic and translate the entire project into Intel DPC++/SYCL code.
* **Windows VRAM Limit Bypass:** Windows artificially caps iGPU memory at 50% of system RAM. Ghost safely edits the Windows Registry (if run as Admin) to unlock maximum VRAM for Core Ultra chips.
* **Smart Crash Interception:** If an AI application attempts to call CUDA and crashes, Ghost intercepts the fatal error and provides the exact `pip install` commands needed to install Intel Extension for PyTorch (IPEX).
* **The Waiting Room TUI:** While your heavy AI models load in the background, Ghost provides an interactive terminal UI featuring live Hardware Polling (Temp/VRAM).
* **Integrated DOOM & Music:** Bored while waiting for a 10GB model to load? Press D to play a fully playable, perfectly scaled version of Chocolate DOOM, or press M to stream royalty-free background music.

## Installation

### Windows
1. Clone or download the repository to your local machine.
2. Open the folder containing the Windows script.
3. Right-click `ghost_intel.ps1` and select **Run with PowerShell**.

*(Linux users: The Bash daemon is currently in beta and will be pushed to the main branch very soon!)*

## Usage

### 1. Enter the Ghost Environment

## Quick Start
To initialize the system:
1. **Open your File Explorer.**
2. **Navigate to the Ghost folder.**
3. **Run the script:** Right-click `ghost_intel.ps1` and select 'Run with PowerShell'.

The environment will automatically resolve the correct directory, activate your Python venv, and apply all Intel/NVIDIA spoofing masks.

*On your first run, the First-Startup Wizard will ask you which AI tool you are using and automatically apply the correct IPEX launch arguments.*

### 2. Launch Your AI Application
Once inside the `(ghost-intel)` terminal, navigate to your AI folder (e.g., SwarmUI or Forge). To launch your script with the Smart Interception and Waiting Room TUI, prefix your command with `ghost-start`:
```bash
ghost-start python launch.py
```

### 3. Translate CUDA Source Code
If you downloaded a custom AI repository that requires CUDA compilation, use the translator tool:
```bash
ghost-translate ./path_to_cuda_project
```
Ghost will scan the folder and output the translated SYCL code into a new `_sycl_out` directory.

### 4. The Waiting Room
While your AI loads, you will see the Ghost TUI. 
* Press **M** to toggle background music.
* Press **D** to launch DOOM. 
* *The exact millisecond your AI finishes loading and opens its web port, Ghost will auto-save DOOM, stop the music, and hand control back to your AI!*

## Verification Test

To confirm that the wrapper is successfully spoofing the hardware identity and linking to Intel XPU, run the following Python diagnostic command inside the ghost environment:

```bash
python -c "import torch, os; import intel_extension_for_pytorch as ipex; print('\n--- SYSTEM DIAGNOSTIC ---\nXPU Available:', torch.xpu.is_available(), '\nHardware Device:', torch.xpu.get_device_name(0) if torch.xpu.is_available() else 'None', '\nSpoofed Identity:', os.getenv('GHOST_SPOOF_NAME'), '\nIPEX Version:', ipex.__version__, '\n-------------------------')"
```

## Troubleshooting & Debugging

**"Running scripts is disabled on this system"**
By default, Windows restricts custom PowerShell scripts. Open PowerShell as Administrator and run:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**"ModuleNotFoundError: No module named 'intel_extension_for_pytorch'"**
Intel does not have a runtime CUDA translator. You must install the native Intel backend for PyTorch. Run:
```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/xpu
pip install intel-extension-for-pytorch
```

**"mkl_intel_thread.dll not found"**
Your system is missing the core Intel math libraries. Please download and install the **Intel oneAPI Base Toolkit**. Ghost will automatically find it and inject the DLLs into your PATH.

**"WARNING: MSVC Compiler (cl.exe) not found"**
You can translate CUDA code to SYCL using `ghost-translate`, but to actually compile the resulting code on Windows, you must install **Visual Studio Build Tools** (specifically the "Desktop development with C++" workload).

**VRAM/Temp stuck on "Loading..."**
Ensure your Intel Arc/Iris display drivers are up to date. Ghost relies on WMI and `dxdiag` to poll hardware stats. If you have a multi-GPU setup, Ghost will automatically sum the VRAM of your Intel cards.

**DOOM looks like a jumbled mess of letters!**
If DOOM looks stretched or unreadable:
1. Change Terminal Size to 80x120
2. Press CTRL + - (Minus) 5 or 6 times to zoom out. Smaller fonts = Higher Resolution ASCII graphics!

**The Music won't play / No Audio**
Ghost uses the native `WMPlayer.OCX` COM object. Ensure Windows Media Player is not completely uninstalled from your Windows Optional Features.

## Technical Architecture
The ghost daemon utilizes the following primary variables and techniques to achieve hardware parity:
* **ONEAPI_DEVICE_SELECTOR**: Forces PyTorch to prioritize `ext_intel_discrete_gpu` over integrated graphics.
* **ZE_MAX_DEVICE_ALLOC_SIZE=0**: Bypasses the Level Zero API single-allocation limits, allowing Core Ultra chips to use 100% of their shared memory for massive LLM tensors.
* **IPEX_XPU_ONEDNN_LAYOUT=1**: Enables highly optimized memory layouts for Wildcat Lake and newer drivers.
* **DPCT / SYCLomatic**: Automates the Ahead-of-Time (AOT) translation of NVIDIA PTX/CUDA kernels into Intel DPC++ via the `ghost-translate` pipeline.

## Disclaimer
This project is for educational and developmental purposes. Hardware spoofing and binary translation may violate the terms of service of certain proprietary applications. Use responsibly.

---

## Ghost-Wrapper: Support the Grind

This tool exists because we shouldn't have to pay the "NVIDIA Tax" to run top-tier AI. If Ghost-Wrapper saved you from a 50-step Linux tutorial or a broken Windows environment, consider supporting the project.

As a solo student dev, I keep things independent. No banks, no trackers - just code and caffeine.

### Digital Tokens (Direct Support)
| Asset | Address |
| :--- | :--- |
| **Ethereum (ETH)** | 0x9e046b3fa85932351b34520837d95bc1ad309748 |
| **Bitcoin (BTC)** | bc1q30wftehky6ct06pshrjh3ekhe9as9uzzdemd0r |

### Social Support
* **Star this Repo:** It is the best way to help the project grow.
* **Report a Bug:** Help me iron out the edge cases.

*Keeping the alternative hardware dream alive, one wrapper at a time.*