# OC-Hackintosh-Z370-E
OpenCore EFI for a machine running i5-8600K, RX 5700 XT, on an ROG Z370-E Board. The whole process was mainly done through Windows. Hopefully this repo helps anyone with a similar setup (or future me) troubleshoot any issues.

## Hardware Configuration
- **Motherboard**: [ROG STRIX Z370-E GAMING](https://rog.asus.com/motherboards/rog-strix/rog-strix-z370-e-gaming-model/)
	- **BIOS Version**: 3005
	- **Ethernet**: Intel I219-V (2)
	- [Manual](http://dlcdnets.asus.com/pub/ASUS/mb/LGA1151/ROG_STRIX_Z370-E_GAMING/E13238_ROG_STRIX_Z370-E_GAMING_UM_WEB_082417.pdf)
- **CPU**: [Intel Core(TM) i5-8600K CPU @ 3.60GHz](https://www.techpowerup.com/cpu-specs/core-i5-8600k.c1948) (*Coffee Lake*)
	- *See [other compatible boards](https://www.intel.com/content/www/us/en/products/sku/126685/intel-core-i58600k-processor-9m-cache-up-to-4-30-ghz/compatible.html)*
- **GPU**: [ASUS ROG STRIX RX 5700 XT GAMING OC](https://www.techpowerup.com/gpu-specs/asus-rog-strix-rx-5700-xt-gaming-oc.b7238)
	- Upgraded from [NITRO+ AMD Radeon RX 580 (8GB) SE](https://www.techpowerup.com/gpu-specs/sapphire-nitro-rx-580-special-edition.b4912) (*See [Other Info](#other-info) for setup*)
- **Memory**: [A-DATA XPG DDR4 2400MHz](https://www.tweaktown.com/reviews/6686/adata-xpg-z1-ddr4-2400-64gb-quad-channel-memory-kit-review/index.html) (4x8GB)
- **Storage**: [Crucial MX500 500GB 3D NAND SATA SSD](https://www.crucial.com/ssd/mx500/ct500mx500ssd1)

## OS Settings
- **SMBIOS**: iMac19,1
- **MacOS Version**: Ventura 13.6.3
- **OpenCore Version**: 1.0.1

# OpenCore
## Getting Started...
- *Pleaseee* go over the [OpenCore guide](https://dortania.github.io/OpenCore-Install-Guide/)
- Make sure you have all the right [**tools**](#tools)
- [Latest BIOS](https://rog.asus.com/motherboards/rog-strix/rog-strix-z370-e-gaming-model/helpdesk_bios/) installed

## Gathering Files
### Firmware Drivers (Universal)
- HfsPlus.efi
- OpenRuntime.efi

### Kexts → `EFI\OC\Kexts`
- **Must Haves**:
	- Lilu
	- VirtualSMC.kext
		- *Plugins*:
			- SMCProcessor.kext
			- SMCRadeonGPU.kext
			- RadeonSensor.kext
			- SMCSuperIO.kext
- **Graphics**:
	- WhateverGreen.kext
- **Audio**:
	- AppleALC.kext
- **Ethernet**:
	- Intel I219-V (2) Network Controller
		- IntelMausi.kext
- USB:
- Extras:

## SSDTs

## Setting up your EFI partition
- Mount the EFI partition using **WinEFIMounter**
	- Select the disk you have macOS installed in
	- Look for a small partition marked `System`
	- Your EFI should be mounted to a new drive letter
- Windows won't let you go through the mounted drive in File Explorer. You can either...
	- Use WinEFIMounter to clone the partition and flush them to the original partition after you've finished setting up, **OR**...
	- Use **Explorer++** (*Run as Admin*) and do your thing through there.

## Config File
> [!NOTE]
> This config has been sanitized. You’ll need to generate your own SMBIOS values using GenSMBIOS to use iCloud, iMessage, or App Store services safely.

- With ProperTree, <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>R</kbd> to perform a clean snapshot
- `config.plist` must match contents of the EFI folder.
- If a file is deleted but left in `config.plist`, OpenCore will stop booting (error).
- Any modifications can benefot from just using the snapshot tool to update `config.plist`

### Config Property List

# BIOS Settings
> [!WARNING]
> PC won't be able to boot to macOS if BIOS settings are not set up properly

BIOS settings for ROG Z370-E (*Refer to OpenCore [Coffee Lake guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#intel-bios-settings)*).

## Disable:
| Setting              | Location/Mode                                             |
|----------------------|-----------------------------------------------------------|
| Fast Boot            | `Boot\Fast Boot`                                          |
| Secure Boot          | `Boot\Secure Boot`                                        |
| Serial/COM Port      | `Advanced\Onboard Devices\Configuration`                  |
| Parallel Port        | *N/A*                                                     |
| VT-d                 | `Advanced\System Agent (SA) Configuration`                |
| CSM                  | `Boot\CSM`                                                |
| Thunderbolt          | *N/A*                                                     |
| Intel SGX            | `Advanced\CPU Configuration`                              |
| Intel Platform Trust | `Advanced\PCH-FW Configuration`<br>↪ **TPM:** *Discrete*  |
| CFG Lock             | `Advanced\CPU Configuration\CPU-Power Management Control` |

## Enable:
| Setting                      | Location/Mode                                                       |
|------------------------------|---------------------------------------------------------------------|
| VT-x                         | `Advanced\CPU Configuration`<br>↪ *Intel Virtualization Technology* |
| Above 4g Decoding            | *N/A*                                                               |
| Hyper-Threading[^1]          | `Advanced\CPU Configuration\Hyper-Threading`                        |
| Execute Disable Bit[^2]        | *N/A*                                                               |
| EHCI/XHCI Hand-off[^3]        | `Advanced\USB Configuration`                                                               |
| OS Type                      | *W8.1/10 UEFI Mode*                                                 |
| DVMT Pre-Allocated (GPU Mem) | *64  MB+*                                                           |
| Sata Mode                    | *AHCI*                                                              |

[^1]: Not supported by i5-8600K, so the option will appeared greyed-out in the BIOS settings.
[^2]: "Data Execution Prevention" is enabled by default (and probably hidden from the UI) for security reasons. Unlikely it's disabled if you're running on a modern system.
	To check:
	1. Open `cmd` and run the command
	```
	wmic OS Get DataExecutionPrevention_SupportPolicy
	```
	2. It will return a value indicating the status of DEP:
	- `0`: DEP is disabled
	- `1`: DEP is enabled for essential Windows programs and services only
	- `2`: DEP is enabled for all programs except those designated by the user
[^3]: Enabled by default in modern OS'. BIOS hands off control automatically as W11 has native USB 3.0 (XHCI) support.

# Features
## Working:
- Ethernet
- All USB Ports
- Time Syncing [*MacOS ↔ W11*]

## Not Working:
- Wi-Fi/Bluetooth

# Issues
Here's a list of issues I ran into trying to set up my build. Linked how I fixed each issue in detail:
- [**Kernel Panic**](Issues/ISSUES.md#kernel-panic-invalid-frame-pointer)
	- Prompts kernel message *"In Memory Panic Stackshot Succeeded"*
	- Fixed using `DevirtualiseMmio`
- [**RTC Write Issues**](.\Issues\ISSUES.md#rtc-write-issues)
	- Prompts boot message *"The system has POSTed in safe mode"*
	- Enabled `DisableRtcChecksum` quirk to fix
- [**Unsynced time after rebooting to Windows 11**](.\Issues\ISSUES.md#unsynced-time-after-rebooting-to-W11)
	- Windows and macOS interpret system time differently
	- Added new registry entry to windows

# Other Info
## Compatibility between *Polaris* and *Navi* GPUs (AMD)
Sometime along the project, I upgraded from an **RX 580** to a **RX 5700 XT** because I was getting horrible performance playing *Dragon's Dogma 2*. After some research, I upgraded to a card that should have just been *plug-and-play*. ***Except***, it wasn't. Luckily, the fix was pretty simple (*See [AMD Boot Arguments](https://dortania.github.io/GPU-Buyers-Guide/misc/bootflag.html#amd-boot-arguments)*).

All I did was **added the boot argument** `agdpmod=pikera`, which is required for all [Navi GPUs](https://en.wikipedia.org/wiki/Radeon_RX_5000_series). Pretty sure that means this method should work for all 5000 and 6000 series cards ([*Don't quote me on that*](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#nvram)).

# Tools
- [**OCSysInfo**](https://github.com/KernelWanderers/OCSysInfo): Obtain detailed hardware information about your system
- [**WinEFIMounter**](https://github.com/franzageek/WinEFIMounter): Mount your Hackintosh EFI partition from Windows
- [**Explorer++**](https://explorerplusplus.com/): GUI to browse through your mounted EFI partition
- [**ProperTree**](https://github.com/corpnewt/ProperTree): Universal `plist` editor
- [**GenSMBIOS**](https://github.com/corpnewt/GenSMBIOS): Generating our SMBIOS data
- [**SSDTTime**](https://github.com/corpnewt/SSDTTime): To create your SSDT if you don't want to just use a [prebuilt one](https://dortania.github.io/Getting-Started-With-ACPI/ssdt-methods/ssdt-prebuilt.html)

# Checklist
- [x] Fix Kernel Panic
- [x] Fix RTC Write Issue
- [x] Fix Time Sync
- [ ] [Fix Sleep](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html)
- [ ] [Add GUI and Boot-chime](https://dortania.github.io/OpenCore-Post-Install/cosmetic/gui.html)
- [ ] ~~Write~~ Finish issues section of README
- [ ] Finish README
- [x] Complete BIOS settings tables
- [ ] Re-configure USB-mapping