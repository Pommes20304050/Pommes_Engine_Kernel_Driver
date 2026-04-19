# Pommes_Engine_Kernel_Driver
Pommes Engine — Personal fork of Cheat Engine 7.5. Fully rebranded from source: all names, strings, binaries, and resources renamed. Custom kernel driver written and compiled from scratch with MSVC/WDK. Main executable processed through VMProtect. Built with Lazarus/FPC. For personal use and local testing only.


# 🍟 Pommes Engine

> A fully rebranded and rebuilt fork of **Cheat Engine 7.5** — every name, string, binary, driver, icon, and resource has been renamed.  
> Based on the official [Cheat Engine 7.5](https://github.com/cheat-engine/cheat-engine) source release by Dark Byte.

---

## 🔁 Complete Rebrand

Every single reference to `Cheat Engine`, `cheatengine`, `DBK`, and `CE` has been replaced throughout the entire codebase and all compiled outputs:

- All **Pascal source files** (`.pas`, `.lpr`, `.lpi`, `.lfm`) — renamed via automated script
- All **compiled binaries** (`.exe`, `.sys`, `.dll`) — strings patched directly in **HxD** in two passes: ANSI encoding and UTF-16 Little Endian (Unicode), since Windows stores strings in both formats
- All **locale/translation files** (`.pot`) — regenerated under the new name
- All **driver files** (`.sys`, `.inf`, `.cer`, `.pdb`) — renamed and recompiled
- All **icons and resources** (`.ico`, `.png`, `.res`) — replaced with new assets
- **Project files** (`.lpi`, `.lps`, `.lpr`, `.res`) — rebuilt under the new project name

The result: no file, string, window title, service name, registry key, or resource in the entire build contains the original Cheat Engine name.

---

## 📋 Overview

Pommes Engine is a full rename and rebuild of Cheat Engine 7.5. Every internal name, string, binary, kernel driver, icon, and locale file has been changed from `cheatengine` / `DBK` to `pommesengine` / `pommes`. The kernel driver was rewritten and recompiled from scratch using Visual Studio, and the final executable was processed through VMProtect.

This project is intended for personal use and local testing only.

---

## 🔧 What Was Changed

### 1. Source Code — Lazarus / Free Pascal

The main application is written in **Free Pascal** and built with the **Lazarus IDE**.

- All Lazarus project files (`.lpi`, `.lpr`, `.lps`) were edited — the project name, output binary name, and internal identifiers were changed from `cheatengine` to `pommesengine`
- The Pascal source file `DBK32functions.pas` was modified to reflect the new driver name (`pommes64`)
- `.pot` locale/translation files were regenerated under the new name

### 2. String Replacement — Script + HxD Hex Editor

Renaming strings happened at **two levels**:

**a) Source level (automated script)**  
A find & replace script was run across all `.pas`, `.lpi`, `.lpr`, `.lfm` source files to swap every occurrence of `CheatEngine`, `cheatengine`, `DBK`, `CE` with the new names.

**b) Binary level (HxD Hex Editor)**  
Strings embedded inside compiled binaries (`.exe`, `.sys`, `.dll`) were edited directly in **HxD** in two encoding passes:

| Pass | Encoding | Why |
|------|----------|-----|
| 1st | **ANSI / Editor encoding** | For narrow-char strings stored as single-byte sequences |
| 2nd | **UTF-16 Little Endian (Unicode)** | For wide-char strings used by Windows APIs and the driver |

Windows stores many internal strings (service names, device names, registry keys) as UTF-16 LE wide strings — both passes are necessary to catch everything.

### 3. Kernel Driver — Self-Written & Recompiled from Scratch

The kernel driver was not simply renamed — it was **rewritten and compiled independently**:

- **Language**: C + inline ASM (`DBKKernel/` source directory)
- **Toolchain**: Microsoft Visual Studio / MSVC + Windows Driver Kit (WDK)
- **Project file**: `DBKKernel.vcxproj` — updated with new driver name and output targets
- **Output**: `pommes64.sys` (91,032 bytes) — a 64-bit Windows kernel driver
- **Build config**: `x64/Releasewithoutsig` — built without a Microsoft-issued code signing certificate
- All intermediate artifacts freshly generated: `apic.obj`, `DBKDrvr.obj`, `DBKFunc.obj`, `memscan.obj`, `debugger.obj`, `ultimap.obj`, `vmxhelper.obj`, and more
- Driver `.inf` files updated with new service name, device name, and driver filename
- A new self-signed certificate `pommes64.cer` (776 bytes) was generated for the driver
- PDB debug symbols `pommes64.pdb` (1.1 MB) included for the driver

> ⚠️ **Note on driver signing:** Windows 64-bit requires kernel drivers to be signed.  
> For local development and testing without a paid EV certificate, use **Test Signing Mode**:
> ```
> bcdedit /set testsigning on
> ```
> Then reboot. This allows loading self-signed drivers locally without disabling Secure Boot.  
> This is the official Microsoft-supported workflow for driver development.

### 4. VMProtect Packing

After compilation, the main executable was run through **VMProtect**.

#### What VMProtect does

VMProtect is a software protection tool that transforms compiled code to make it significantly harder to reverse engineer:

- **Virtualization** — Selected code sections are converted into custom virtual machine bytecode. Instead of running native x86-64 instructions, the code runs inside an embedded VM interpreter with its own unique instruction set, different per build.
- **Mutation** — Code is rewritten into functionally equivalent but structurally different sequences, breaking pattern-based static analysis.
- **Packing & compression** — The final binary is compressed and unpacks itself at runtime, which is why the `.vmp.exe` files (~6.7 MB) are significantly smaller than the originals (~16.8 MB).
- **Anti-debug / Anti-dump** — The protected build includes checks that detect debuggers, memory dumpers, and analysis tools at runtime.

The clean unprotected build is preserved alongside the packed version for debugging purposes.

| File | Size | Notes |
|------|------|-------|
| `pommesengine-x86_64.exe` | ~16.8 MB | Clean compiled build |
| `pommesengine-x86_64.exe.bak` | ~16.8 MB | Backup before packing |
| `pommesengine-x86_64.vmp.exe` | ~6.7 MB | VMProtect-packed output |
| `pommesengine-x86_64-SSE4-AVX2.exe` | ~16.8 MB | AVX2-optimized variant |
| `pommesengine-x86_64-SSE4-AVX2.vmp.exe` | ~6.7 MB | AVX2 + VMProtect |

Debug symbols (`.dbg`, ~148 MB) were preserved alongside the builds.

### 5. Icons & Assets

- `cheatengine.ico` → `pommes_engine.ico` + `pommes_engine_128.ico`
- PNG variants (`pommes_engine.png`, `pommes_engine_128.png`) added
- `.res` resource file recompiled with new icon

---

## 📁 File Structure (Key Changes)

```
Cheat Engine/
├── bin/
│   ├── pommes64.sys              ← renamed + recompiled kernel driver
│   ├── pommes64.cer              ← new certificate
│   ├── pommes64.pdb              ← debug symbols for driver
│   ├── pommesengine-x86_64.exe   ← main application
│   ├── pommesengine-x86_64.vmp.exe  ← VMProtect-packed
│   └── languages/
│       └── pommesengine-x86_64.pot  ← renamed locale file
├── dbk32/
│   └── DBK32functions.pas        ← modified source
DBKKernel/
├── DBKKernel.vcxproj             ← VS project, renamed targets
├── pommes64.sys.recipe           ← updated build recipe
└── x64/Releasewithoutsig/        ← fresh build output
    ├── pommes64.sys
    ├── pommes64.Build.CppClean.log
    └── [all .obj, .tlog, .pdb artifacts]
```

---

## 🛠️ Building from Source

### Prerequisites

- [Lazarus IDE](https://www.lazarus-ide.org/) + Free Pascal Compiler (FPC)
- Visual Studio 2019+ with Windows Driver Kit (WDK) for the kernel driver
- VMProtect (optional, for packing)

### Build Steps

```bash
# 1. Open cheatengine.lpi in Lazarus IDE
# 2. Build → Build All

# For the kernel driver (Visual Studio):
# Open DBKKernel/DBKKernel.sln
# Select x64 / Releasewithoutsig
# Build Solution

# To load the driver locally (Test Signing Mode):
bcdedit /set testsigning on
# Reboot, then install via the .inf file
```

---

## ⚠️ Disclaimer

This project is based on [Cheat Engine](https://github.com/cheat-engine/cheat-engine) by Dark Byte, which is licensed under the [GPL-2.0 license](https://github.com/cheat-engine/cheat-engine/blob/master/LICENSE.md).

- For personal and local testing use only
- Do not use in online multiplayer games — violates terms of service
- The kernel driver runs at ring-0 — use in a VM or test machine if unsure

---

## 📎 Credits

- Original project: [Cheat Engine by Dark Byte](https://github.com/cheat-engine/cheat-engine)
- Build toolchain: Lazarus IDE, Free Pascal, Microsoft Visual Studio / WDK
- Packing: VMProtect
- Hex editing: [HxD](https://mh-nexus.de/en/hxd/)
