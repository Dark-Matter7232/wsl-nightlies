<div align="center">
  <h1>🌙 WSL Nightly Builds</h1>
  <p><b>Automated Windows Subsystem for Linux (WSL) builds for both Snapdragon/ARM64 and x64 targets, including MSI installers and optimized kernel workflows.</b></p>
  
  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
  [![Snapdragon X Optimized](https://img.shields.io/badge/Optimized_for-Snapdragon_X-blue?logo=qualcomm)](https://www.qualcomm.com/products/mobile/snapdragon)
  [![Zen 4 Optimized](https://img.shields.io/badge/Optimized_for-Zen_4-blue)](#-zen-4-optimizations)
</div>

---

## 🚀 What this does

This repository contains GitHub Actions workflows for both **Snapdragon/ARM64** and **x64** WSL builds.

### 1. 📦 MSI workflows
* **Build WSL ARM64 MSI (Nightly):** Builds the upstream `microsoft/WSL` ARM64 MSI package.
* **Build Dark-Matter7232 WSL MSI (Nightly):** Builds both ARM64 and x64 installers from `Dark-Matter7232/WSL` in parallel. ARM64 and x64 use architecture-specific optimization settings.

### 2. ⚡ Snapdragon / ARM64 kernel workflow
* **Build WSL Kernels (Nevuly Rolling):** Builds ARM64 and x64 Nevuly rolling kernels in parallel from one matrix workflow.
* The ARM64 variant uses GCC 14 cross-compilation and Snapdragon-oriented tuning. Both variants enable DRM/VGEM, USB/IP VHCI, and USB serial CH341 support.

### 3. ⚡ x64 kernel workflow
* The x64 matrix variant uses the Nevuly x86 WSL config, Zen 4 tuning, and produces a `bzImage` artifact for x64 WSL use.

### 5. 🔥 x64 / Zen 4 tuning
* The x64 workflows apply `-march=x86-64-v4 -mtune=znver4` for Linux-side tuning.
* Windows binaries use MSVC x64 code generation with `/favor:AMD64` as the closest scheduling hint available.

---

## ⬇️ Downloading builds

The Dark-Matter WSL workflow publishes a dated **pre-release** containing both MSI installers and their build-info files whenever both architecture builds succeed. The Nevuly kernel workflow also publishes a dated pre-release containing both kernel builds. Browse the [nightly releases](../../releases) and download the ARM64 or x64 package for the matching system. MSI and kernel releases use separate tags. If a build is rerun on the same day, it updates that day's release.

For the Microsoft WSL MSI, go to the [**Actions**](../../actions) tab, select the workflow, then download the artifact from a successful run.

> **💡 Tip:** The generated artifacts are compressed archives containing:
> * **For ARM64 MSI:** typically `wsl-arm64-<date>-<commit>.msi` + `build-info.txt`
> * **For x64 MSI:** typically `wsl-x64-<date>-<commit>.msi` + `build-info.txt`
> * **For ARM64 kernel:** `Image-arm64` + `config-arm64-nevuly-base` + `build-info.txt`
> * **For x64 kernel:** `bzImage-x64` + `config-x64-nevuly-base` + `build-info.txt`

---

## 🎯 Triggering a build manually

You can trigger a build on-demand from the Actions tab → **[Workflow Name]** → **Run workflow**.

### ⏱️ Schedule
* **MSI workflows:** Daily at **06:00 UTC**. The Dark-Matter workflow skips scheduled builds when the upstream commit has not changed.
* **ARM64 Nevuly Rolling Kernel:** Weekly on Sunday at **00:00 UTC**.
* **x64 Nevuly Rolling Kernel:** Weekly on Sunday at **00:00 UTC**.

---

## 📦 What's included in the MSI builds

The MSI workflows produce full-featured WSL installers. Exact output naming differs by target architecture, but the packaged contents are broadly the same:

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

## 🔥 Optimization profiles

This repo carries two optimization tracks depending on the workflow target.

### Snapdragon X / ARM64

Builds heavily target **Qualcomm Snapdragon X Elite/Plus** as the ARM64 baseline.

| Component | Flag | Effect |
|-----------|------|--------|
| **Windows binaries** (wsl.exe, etc) | MSVC `/arch:armv8.2` | Enables newer ARM64 ISA features used by Snapdragon-targeted builds |
| **Linux init binary** | Clang `-march=armv8.4-a+crypto+dotprod+fp16fml` | Targets a modern ARM64 baseline with crypto and SIMD features |
| **Linux Nevuly Rolling Kernel** | GCC `-O3 -march=armv8.7-a+crypto -mtune=cortex-x4` | Tunes the kernel for Snapdragon X-class ARM64 hardware |

### Zen 4 / x64

Builds heavily target **AMD Zen 4** systems as the x64 baseline.

### ⚙️ What's optimized

| Component | Flag | Effect |
|-----------|------|--------|
| **Windows binaries** (wsl.exe, etc) | MSVC `/favor:AMD64` | Uses AMD64 scheduling preferences for x64 builds |
| **Linux init binary** | Clang `-march=x86-64-v4 -mtune=znver4` | Targets the x86-64-v4 feature level and tunes code generation for Zen 4 |
| **Linux Nevuly Rolling Kernel** | GCC `-O3 -march=x86-64-v4 -mtune=znver4` | Highly tuned scheduling and ISA targeting for Zen 4 workloads |

### 📈 Key performance features
* **Snapdragon ARM64 path:** Uses newer ARM64 architecture features and Snapdragon-oriented tuning for supported devices.
* **x86-64-v4 baseline:** Enables a modern x64 ISA baseline including AVX2-class features and newer vector capabilities expected on recent CPUs.
* **Zen 4 tuning:** Improves instruction selection and scheduling for AMD Zen 4 microarchitecture.
* **`-O3` kernel build:** Pushes more aggressive inlining and loop optimizations for hot kernel code paths.
* **Modern SIMD paths:** Benefits compression, crypto, checksum, and other throughput-sensitive workloads.

> **⚠️ Compatibility Note:** ARM64 artifacts target newer Snapdragon-class devices, and x64 artifacts target an `x86-64-v4` / Zen 4-style baseline. Older CPUs outside those feature levels are out of scope for the optimized builds. Pre-built NuGet components (kernel, WSLg system VHD, DirectX libs) ship as-is from Microsoft and are not affected by these flags.

---

## 🛠️ Build details

| Setting | Value |
|---------|-------|
| **Sources** | [microsoft/WSL](https://github.com/microsoft/WSL), [Dark-Matter7232/WSL](https://github.com/Dark-Matter7232/WSL), [Nevuly/WSL2-Linux-Kernel-Rolling](https://github.com/Nevuly/WSL2-Linux-Kernel-Rolling) |
| **Platforms** | ARM64 and x64 |
| **Configuration** | Release |
| **Code signing** | Skipped (unsigned dev build) |
| **Windows SDK** | 10.0.26100.0 |
| **Ccache** | 5GB Limit / `$GITHUB_WORKSPACE/.ccache` |

---

## 📝 Notes & License

* These are **unofficial, unsigned** builds from the public WSL source code.
* Nightly releases remain available under [Releases](../../releases); workflow artifacts expire after **90 days**.
* WSL is licensed under the [MIT License](https://github.com/microsoft/WSL/blob/master/LICENSE) by Microsoft. This repository only contains the build automation workflow.
