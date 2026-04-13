# eSim 2.5 Installation on Ubuntu 25.04 – Issue Analysis & Fixes

## 📌 Overview
This project involves installing **eSim 2.5** on **Ubuntu 25.04 (latest non-LTS release)** and systematically identifying and resolving dependency and script-related issues.

Ubuntu 25.04 introduces changes in package availability and system structure, leading to multiple failures in the original installation script.

The `nghdl.zip` package has been modified and included in this branch to ensure compatibility.

Additionally, the `/src`, `/images`, and `/library` directories are bundled to enable a fully self-contained installation without external dependencies.

## 🚀 Key Contributions

### Installer Improvements
- Adapted eSim installer for Ubuntu 25.04 compatibility
- Replaced deprecated/unsupported package installations
- Migrated KiCad installation from APT to Snap

### Compatibility Fixes
- Fixed KiCad 8 directory structure and permissions
- Resolved SKY130 + volare execution issues
- Implemented LLVM 16 build from source (critical fix)
- Fixed GHDL build by linking correct LLVM version

### Packaging Enhancements
- Made installer self-contained by bundling required resources
 
## 📦 This repository includes:
- Issues encountered during installation
- Root cause analysis
- Fixes implemented in the installer script
- Final working setup

---

## 🎯 Objective
- Enable successful installation of eSim 2.5 on Ubuntu 25.04
- Identify and analyze installation failures
- Modify the installer script for compatibility
- Resolve critical dependency issues (LLVM, GHDL)

---

## 🛠️ Environment Details
- OS: Ubuntu 25.04 (VM)
- RAM: 32GB
- vCPU: 8
- Shell: Bash
- Tools Used: Git, Snap, APT

---

## 📂 Repository Structure
```
.
└── Ubuntu/
    ├── images/
    ├── install-eSim-scripts/
    ├── library/
    ├── nghdl/
    ├── src/
    ├── install-eSim.sh
    └── nghdl.zip
    
```

---

## 🚀 Installation Process
```bash

git clone https://github.com/Jashan32/eSim.git
cd eSim
git checkout installers
cd Ubuntu
chmod +x install-eSim.sh
./install-eSim.sh --install

```
---

## 🐞 Issues & Fixes

### 1. Unsupported Ubuntu version

**Error:** 
``` bash 
Detected Ubuntu Version: 
Unsupported Ubuntu version: 25.04 ()
```

**Cause:** `install-eSim.sh` uses a case statement based on Ubuntu version and it opens its corresponding version's file, but for 25.04 no such file exist so it fallback to this error message.

**Fix:** create `install-eSim-25.04.sh` and link it to new case for version 25.04.

### 2. Missing `lsb-release`
**Issue:** Script failed during Ubuntu version detection.  
**Error:**
``` bash
./install-eSim.sh: line 26: lsb_release: command not found
```
**Fix:**

``` bash
sudo apt install -y lsb-release
```
**Change Made:** Added installation before usage in script.

### 3. Incorrect xz-utils Installation Command
**Issue:** Script used incorrect syntax (apt-get xz-utils).  
**Error:** 
``` bash
E: Invalid operation xz-utils
```
**Fix:**

``` bash
~~apt-get xz-utils~~
sudo apt-get install -y xz-utils
```
**Change Made:** Corrected package name and command format.

### 4. KiCad Installation Failure (APT → SNAP Migration)
**Issue:** APT installation failed due to repository/version mismatch on Ubuntu 25.04.  
 **Fix:**
``` bash
sudo snap install kicad --classic
```
**Change Made:** Replaced APT-based installation with Snap for stability.

### 5. KiCad Library Path Incompatibility (KiCad 8)
**Issue:** Script was using old KiCad 6 directory structure (~/.config/kicad/6.0).  
**Fix:**
Updated paths to:
```
~/.config/kicad/8.0
~/Documents/KiCad/8.0/symbols
```
Removed system-wide writes (/usr/share/kicad) and switched to user directories.

**Change Made:** Ensured compatibility with KiCad 8 structure and permissions.

### 6. Permission Issues in SKy130 & Volare Setup
**Issue:** volare commands failed under sudo due to missing PATH resolution.  
**Fix:**
``` bash
sudo $(which volare) enable --pdk sky130 --pdk-root /usr/share/local/
```
**Change Made:** Explicit binary path used with sudo + ensured directory handling.


### 7. Missing Desktop Directory & Shortcut Issues
**Issue:** Script assumed ~/Desktop exists and failed during shortcut creation.  
**Fix:**
``` bash
mkdir -p $HOME/Desktop
```
**Change Made:** Added directory existence check + improved permission handling.


### 8. Missing software-properties-common
**Issue:** Required for repository management but not installed.  
**Fix:**
``` bash
sudo apt-get install -y software-properties-common
```


### 9. LLVM Version Mismatch (Critical Fix)
**Issue:** Script relied on system LLVM (v11), which caused GHDL build failures.  
**Fix Implemented:** Built LLVM 16 from source.

``` bash
install_llvm16() {
  set -e

  sudo apt update
  sudo apt install -y build-essential cmake ninja-build python3 git

  if [ ! -d "llvm-project" ]; then
    git clone https://github.com/llvm/llvm-project.git
  fi

  cd llvm-project
  git fetch
  git checkout llvmorg-16.0.6

  mkdir -p build && cd build

  cmake -G Ninja ../llvm \
    -DLLVM_ENABLE_PROJECTS="clang" \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm-16 \
    -DLLVM_TARGETS_TO_BUILD="X86"

  ninja -j$(nproc)
  sudo ninja install

  export LLVM_CONFIG=/opt/llvm-16/bin/llvm-config
}
```
**Impact:** This resolves incompatibility with outdated LLVM (v11) and enables successful GHDL compilation on modern Ubuntu systems.  
**Result:** Ensured modern LLVM compatibility required by GHDL.


### 10. GHDL Build Failure (Linked to LLVM)
**Issue:** GHDL was configured with incorrect LLVM path.  
**Fix:**
``` bash
./configure --with-llvm-config=/opt/llvm-16/bin/llvm-config
```
**Change Made:** Updated installGHDL() to use LLVM 16 path.

---

## 📊 Summary
Issues Found: 10  
Issues Fixed: 10   
Critical Fixes: LLVM + GHDL  

---

## 📌 Conclusion
The original eSim installer was incompatible with Ubuntu 25.04 due to outdated dependencies and structural changes in the OS.

After systematic debugging and targeted fixes, the installer now runs reliably on modern Ubuntu systems, with improved stability and self-contained setup.