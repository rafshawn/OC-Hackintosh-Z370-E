# OC-Hackintosh-Z370-E
OpenCore EFI for a machine running i5-8600K, RX 5700 XT, on an ROG Z370-E Board

# Hardware Configuration
- **Motherboard**: [ROG STRIX Z370-E GAMING](https://rog.asus.com/motherboards/rog-strix/rog-strix-z370-e-gaming-model/)
	- **Ethernet**: Intel I219-V (2)
	- [Manual](http://dlcdnets.asus.com/pub/ASUS/mb/LGA1151/ROG_STRIX_Z370-E_GAMING/E13238_ROG_STRIX_Z370-E_GAMING_UM_WEB_082417.pdf)
- **CPU**: [Intel Core(TM) i5-8600K CPU @ 3.60GHz](https://www.techpowerup.com/cpu-specs/core-i5-8600k.c1948) (*Coffee Lake*)
	- *See [other compatible boards](https://www.intel.com/content/www/us/en/products/sku/126685/intel-core-i58600k-processor-9m-cache-up-to-4-30-ghz/compatible.html)*
- **GPU**: [ASUS ROG STRIX RX 5700 XT GAMING OC](https://www.techpowerup.com/gpu-specs/asus-rog-strix-rx-5700-xt-gaming-oc.b7238)
	- Upgraded from [NITRO+ AMD Radeon RX 580 (8GB) SE](https://www.techpowerup.com/gpu-specs/sapphire-nitro-rx-580-special-edition.b4912) (*See [Other Info](#other-info) for setup*)
- **Memory**: [A-DATA XPG DDR4 2400MHz](https://www.tweaktown.com/reviews/6686/adata-xpg-z1-ddr4-2400-64gb-quad-channel-memory-kit-review/index.html) (4x8GB)
- **Storage**: [Crucial MX500 500GB 3D NAND SATA SSD](https://www.crucial.com/ssd/mx500/ct500mx500ssd1)

# OS Settings
- **SMBIOS**: iMac19,1
- **MacOS Version**: Ventura 13.6.3

# OpenCore
## Config File

## Setting up your EFI partition
- Mount the EFI partition using **WinEFIMounter**
	- Select the disk you have macOS installed in
	- Look for a small partition marked `System`
	- Your EFI should be mounted to a new drive letter
- Windows won't let you go through the mounted drive in File Explorer. You can either...
	- Use WinEFIMounter to clone the partition and flush them to the original partition after you've finished setting up, **OR**...
	- Use **Explorer++** (*Run as Admin*) and do your thing through there.

# BIOS Settings
BIOS settings for ROG Z370-E. *(Refer to OpenCore [Coffee Lake guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#intel-bios-settings))*

> [!IMPORTANT]
> PC won't be able to boot to macOS if BIOS settings are not set up properly

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
| Intel Platform Trust | `Advanced\PCH-FW Configuration`<br>↪ **TPM:** *Discrete*        |
| CFG Lock             | `Advanced\CPU Configuration\CPU-Power Management Control` |

## Enable:
| Setting                      | Location/Mode                                                     |
|------------------------------|-------------------------------------------------------------------|
| VT-x                         | `Advanced\CPU Configuration`<br>↪ *Intel Virtualization Technology* |
| Above 4g Decoding            | *N/A*                                                             |
| Hyper-Threading              |                                                                   |
| Execute Disable Bit          |                                                                   |
| EHCI/XHCI Hand-off           |                                                                   |
| OS Type                      | *W8.1/10 UEFI Mode*                                               |
| DVMT Pre-Allocated (GPU Mem) | *64  MB+*                                                         |
| Sata Mode                    | *AHCI*                                                            |

# Features
## Working:
- Ethernet
- All USB Ports
- Time Syncing [*MacOS ↔ W11*]

## Not Working:
- Wi-Fi/Bluetooth

# Issues
## Kernel Panic (`Invalid frame pointer`)
> [!NOTE]  
> This really depends on your setup. Refer to OpenCore [Troubleshooting guide](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/kernel-issues.html#kernel-panic-on-invalid-frame-pointer) for a detailed fix.

### Using `DevirtualiseMMIO`
This fix requires abit of trial and error. 

## RTC Write Issues
- Prompts boot message *"The system has POSTed in safe mode"* after rebooting from MacOS
- Refer to *[Fixing RTC write issues (guide)](https://dortania.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)* for more info

## Unsynced time after rebooting to W11
One annoying issue is that your clock changes every time you boot to Windows after using macOS. The time is essentially stored on the motherboard, but Windows and macOS interpret this stored time differently.
- Windows does not apply a timezone to the system time
- macOS interprets the system time as [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)

As a result of this, neither will agree with each other and the time displayed on Windows will always be messed up after rebooting from macOS. The way to fix this is to ***change how Windows interprets time*** as UTC.

### Quick fix
1. Download <file_name.reg> (Right click > Save link as...)
2. Run as Administrator
3. Restart

### Manual fix
1. Launch the Registry Editor (<kbd>⊞ Win</kbd>+<kbd>R</kbd>, type `regedit`)
2. Go to
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```
3. Right click > New > `DWORD (32-bit)`
4. Name the key `RealTimeIsUniversal` and set the value data to `1`
5. Save and Restart

### Disable fix
1. Launch `regedit`
2. Go to
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```
3. Delete the key `RealTimeIsUniversal`

# Other Info
## Compatibility between *Polaris* and *Navi* GPUs (AMD)
Sometime along the project, I upgraded from an **RX 580** to a **RX 5700 XT** because I was getting horrible performance playing *Dragon's Dogma 2*. After some research, I upgraded to a card that should have just been *plug-and-play*. ***Except***, it wasn't. Luckily, the fix was pretty simple (*Refer to [AMD Boot Arguments](https://dortania.github.io/GPU-Buyers-Guide/misc/bootflag.html#amd-boot-arguments)*).

All I did was **added the boot argument** `agdpmod=pikera`, which is required for all [Navi GPUs](https://en.wikipedia.org/wiki/Radeon_RX_5000_series). Pretty sure that means this method should work for all 5000 series cards (*Don't quote me on that*).

# Tools
- [**OCSysInfo**](https://github.com/KernelWanderers/OCSysInfo):
- [**WinEFIMounter**](https://github.com/franzageek/WinEFIMounter): 
- [**Explorer++**](https://explorerplusplus.com/):

# Checklist
- [x] Fix Kernel Panic
- [ ] Fix RTC Write Issue
- [x] Fix Time Sync
- [ ] Fix Sleep
- [ ] [Add GUI and Boot-chime](https://dortania.github.io/OpenCore-Post-Install/cosmetic/gui.html)
- [ ] ~~Write~~ Finish issues section of README
- [ ] Finish README
- [ ] Complete BIOS settings tables
