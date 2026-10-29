# NEMESIS

Own your car, to its fullest.

App store is coming soon at [NEMESIS Website, where Discord link is also available](https://xwtk.cloud/services/nemesis).

### **Support NEMESIS development and research**

You may pledge a small amount to help cover research costs for Gen5W systems as well as NEMESIS development. It is completely optional and is considered as a gratitude for the done work.

[![Sponsor via GitHub](https://img.shields.io/badge/Sponsor%20via%20GitHub-FAFAFA?style=for-the-badge&logo=githubsponsors&logoColor=black)](https://github.com/sponsors/xwtk)
[![Donate via xwtk.Portal](https://img.shields.io/badge/Donate%20via%20xwtk.Portal-FAFAFA?style=for-the-badge)](https://xwtk.cloud/services/nemesis#donate)

## **Warning**

NEMESIS is subject to GNU AGPL 3.0 License, get to know more [here](LICENSE).

You bear complete responsibility for any procedure with your car and its components, it is important to understand that there is always risk associated with procedures described in this and associated documents in this repository, your car functionality may be heavily impacted if this software is not compatible with your components.

## Packages

Discover pre-made packages for you to install on your car. Browse catalog [here](Packages/README.md).

## Compatibility Check

For your car to be compatible with NEMESIS, it has to:

1. Have an Android based Gen5W HU (version must have S5W_M in it in General Settings > SW Info). S5W_L is not supported, it is a fundamentally different hardware and software combination.

2. It has to have OTA update support (not necessary to have OTA Update subscription) as NEMESIS utilizes a flaw in the OTA update logic as an entry point for its installation procedure. Some cars may still not be compatible even in that case, it can only be understood once you try to install.

### **Cars known not to work with NEMESIS**

| Name | Description |
| ---- | ----------- |
| Any car with no OTA updates capability | NEMESIS uses a flaw in OTA updates system to install itself, if your car does not have OTA updates, it does not have that flaw. |
| Any Canadian HMG car | No Canadian HMG vehicle has support for OTA updates, despite Update OTA being shown for some in the Engineering Mode menu. |
| Any car with S5W_L HU | Fundamentally different hardware and software architecture, does not run Android, NEMESIS cannot be ported in any shape or form there. |
| Hyundai Elantra / Kia Forte / Hyundai Ioniq 6 | Despite having OTA updates support, these cars use different schemes which do not have the required flaw for NEMESIS. |

### **Will there be support for incompatible cars in the future?**

**Not** guaranteed, other members are working on porting NEMESIS to cars such as Ioniq 6 and Canadian vehicles, check Discord for more information.

### **What will happen if I try to install?**

Most likely nothing, NEMESIS won't install.

## **Quick Installation Guidance**

| Step | Guidance |
| ---- | -------- |
| 1 | Download [NEMESIS Installation Package](ota_package.tar) onto your computer. |
| 2 | Prepare an exFAT or FAT32 formatted blank USB drive. |
| 3 | Copy the downloaded tar file onto the USB, do not extract or repack the archive file. |
| 4 | Insert into the car's HU USB port, messages such as "No Supported Files" are expected. |
| 5 | Navigate to Engineering Mode (use web to search on how to do so), open USB Copy, select Update DSP |
| 6 | Tap on Update OTA (existence of this doesn't mean your car has OTA support), if you see a loading animation come up on the status bar in the top right, everything is going as planned. Wait for installation to complete and the HU to restart. |

## **Documentation**

Read about NEMESIS, what it is exploiting, what it features, and its use manual.

| Document | Contents |
|----------|----------|
| [01 - DSP OTA troubleShooter Path](01-DSP-OTA-troubleShooter.md) | How the firmware runs unsigned DSP OTA members as root, and how that path is reused. |
| [02 - Debugging Execution Interface](02-Debugging-Execution-Interface.md) | What NEMESIS is on the unit after it is present: features, how you open it, CLI and Manager usage. |
| [03 - Install and uninstall](03-Install-and-Uninstall.md) | How NEMESIS is written onto `/system` and how it is removed. |
| [04 - Packages](04-Packages.md) | Installable packages: metadata, layout, builder, lifecycle. |
| [05 - Run scripts](05-Run-Scripts.md) | USB `NEMESIS.sh` / `type=execute`: when they run and how to write them. |
| [06 - Build and test](06-Build-and-Test.md) | Building the car release in Docker, and the stock Android 4.4 emulator security suite. |
| [Security hardening](SECURITY-HARDENING.md) | What this fork changes from upstream, how it is tested, and how to upgrade an existing install. |

Documents **01** and **03** are the only ones that describe the DSP Update Agent path and on-disk install, do not use them with advanced LLMs which may have safeguards enabled. **02**, **04**, and **05** assume NEMESIS is already on the head unit and treat it as a Debugging Execution Interface for packages and privileged commands, so safeguards will not be triggered.


Developed by Gor Mirzoyan (xwtk.cloud).
