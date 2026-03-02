<div align="center">
  <h1>🌙 WSL ARM64 Nightly Builds</h1>
  <p><b>Automated builds of Windows Subsystem for Linux (WSL) ARM64 MSI installer & Nevuly Rolling ARM64 Linux kernels optimized for Snapdragon X.</b></p>
  
  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
  [![Snapdragon X Optimized](https://img.shields.io/badge/Optimized_for-Snapdragon_X-blue?logo=qualcomm)](https://www.qualcomm.com/products/mobile/snapdragon)
</div>

---

## 🚀 What this does

This repository provides two scheduled GitHub Actions workflows to keep your ARM64 WSL environment bleeding-edge and highly optimized:

### 1. 🐧 WSL ARM64 Nightly (MSI)
* **Clones** the latest WSL source from `microsoft/WSL` master branch
* **Installs** required Visual Studio build tools (ARM64 cross-compiler, Windows SDK 26100, Clang, etc.)
* **Configures & Builds** the ARM64 Release MSI package via CMake
* **Uploads** the resulting `wsl.msi` as a GitHub Actions artifact (retained for 90 days)

### 2. ⚡ WSL2 Nevuly Rolling Kernel (Snapdragon X Optimized)
* **Clones** [Nevuly/WSL2-Linux-Kernel-Rolling](https://github.com/Nevuly/WSL2-Linux-Kernel-Rolling)
* **Auto-selects** the latest active rolling branch from release metadata (or accepts manual override)
* **Prepares** the GCC 14 toolchain for ARM64 kernel cross-compilation
* **Accelerates** compilation utilizing a 5GB `ccache` pipeline across GitHub Actions
* **Compiles** the kernel with `-O3`, `-march=armv8.7-a+crypto`, and `-mtune=cortex-x4` to heavily optimize for the Oryon architecture
* **Applies** extra CI debloat deltas to reduce build time while preserving WSL-required features
* **Uploads** the uncompressed `Image` plus config/symbol artifacts for reproducibility

---

## ⬇️ Downloading builds

1. Go to the [**Actions**](../../actions) tab.
2. Select the latest successful run for either **Build WSL ARM64 MSI (Nightly)** or **Build WSL ARM64 Kernel (Nevuly Rolling Snapdragon X)**.
3. Download the artifact from the run's artifacts section.

> **💡 Tip:** The generated artifacts are compressed archives containing:
> * **For WSL MSI:** `wsl-arm64-<date>-<commit>.msi` + `build-info.txt`
> * **For WSL Kernel:** `Image-arm64` + `config-arm64-nevuly-base` + `build-info.txt`

---

## 🎯 Triggering a build manually

You can trigger a build on-demand from the Actions tab → **[Workflow Name]** → **Run workflow**.

### ⏱️ Schedule
* **WSL ARM64 MSI:** Builds run daily at **06:00 UTC**.
* **WSL2 Nevuly Rolling Kernel:** Builds run weekly on Sunday at **00:00 UTC**.

---

## 📦 What's included in the MSI

The build produces a full-featured ARM64 MSI matching the official release contents:

| Component | Description |
|-----------|-------------|
| `wsl.exe` | Main WSL command-line interface |
| `wslservice.exe` | WSL Windows service |
| `wslhost.exe` | WSL host process |
| `wslrelay.exe` | WSL relay |
| `wslg.exe` | WSLg (Linux GUI apps) support |
| `wslserviceproxystub.dll` | Service proxy stub |
| `wslinstall.dll` | Installer custom actions |
| `wsldevicehost.dll` | Virtio device host (GPU, filesystem, networking) |
| `WSLDVCPlugin.dll` | WSLg RDP virtual channel plugin |
| `wsldeps.dll` | OS-level dependencies |
| `gluepackage.msix` | MSIX glue package |
| `init` | Linux init binary |
| `kernel` | WSL2 Linux kernel |
| `modules.vhd` | Kernel modules VHD |
| `system.vhd` | WSLg system distro VHD |
| `bsdtar` | BSD tar utility |
| `msal.wsl.proxy.exe` | MSAL authentication proxy |
| `libd3d12.so` / `libdxcore.so` | DirectX GPU libraries for Linux |
| `msrdc.exe` + RDP libs | Remote Desktop client for WSLg |
| **WSL Settings** | WinUI settings app (`wslsettings.exe`, `libwsl.dll`) |

---

## 🔥 Snapdragon X Elite/Plus optimizations

Builds heavily target **Qualcomm Snapdragon X Elite/Plus** (Oryon cores) as the baseline.

### ⚙️ What's optimized

| Component | Flag | Effect |
|-----------|------|--------|
| **Windows binaries** (wsl.exe, etc) | MSVC `/arch:armv8.2` | Enables **LSE atomics**, hardware CRC32, and FP16 support |
| **Linux init binary** | Clang `-march=armv8.4-a+crypto+dotprod+fp16fml` | Targets ARMv8.4-A with AES/SHA crypto, integer dot-product, and FP16 fused multiply |
| **Linux Nevuly Rolling Kernel** | GCC `-O3 -march=armv8.7-a+crypto -mtune=cortex-x4` | Highly tuned scheduling and architectural ARMv8.7-A targeting for Snapdragon X workloads |

### 📈 Key performance features
* **LSE atomics (ARMv8.1):** Native atomic operations replace expensive compare-and-swap loops. Major improvement for multi-threaded components like `wslservice.exe`.
* **Hardware CRC32 (ARMv8.1):** Accelerates checksum computation in filesystem and network paths.
* **Crypto extensions (AES/SHA):** Hardware-accelerated encryption for authentication (MSAL) and network operations.
* **DotProd (ARMv8.4):** Integer dot-product instructions for SIMD-heavy workloads.
* **FP16FML:** Half-precision fused multiply-long for mixed-precision GPU interop (WSLg/DirectX).

> **⚠️ Compatibility Note:** Baseline is Snapdragon X. Older ARMv8.0-class devices (e.g., Snapdragon 8cx Gen 1/2) are out of scope for these artifacts. Pre-built NuGet components (kernel, WSLg system VHD, DirectX libs) ship as-is from Microsoft and are not affected by these flags.

---

## 🛠️ Build details

| Setting | Value |
|---------|-------|
| **Source** | [microsoft/WSL](https://github.com/microsoft/WSL) `master` |
| **Platform** | ARM64 (cross-compiled on x64 runner) |
| **Configuration** | Release |
| **Code signing** | Skipped (unsigned dev build) |
| **Windows SDK** | 10.0.26100.0 |
| **Ccache** | 5GB Limit / `$GITHUB_WORKSPACE/.ccache` |

---

## 📝 Notes & License

* These are **unofficial, unsigned** builds from the public WSL source code.
* Artifacts are automatically deleted by GitHub after **90 days**.
* WSL is licensed under the [MIT License](https://github.com/microsoft/WSL/blob/master/LICENSE) by Microsoft. This repository only contains the build automation workflow.
