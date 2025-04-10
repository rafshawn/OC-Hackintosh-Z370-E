# Hardware Configuration
- **Motherboard**: [ROG STRIX Z370-E GAMING](https://rog.asus.com/motherboards/rog-strix/rog-strix-z370-e-gaming-model/)
	- **BIOS Ver.**: 3005
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
## Setting up your EFI partition
- Mount the EFI partition using **WinEFIMounter**
	- Select the disk you have macOS installed in
	- Look for a small partition marked `System`
	- Your EFI should be mounted to a new drive letter
- Windows won't let you go through the mounted drive in File Explorer. You can either...
	- Use WinEFIMounter to clone the partition and flush them to the original partition after you've finished setting up, **OR**...
	- Use **Explorer++** (*Run as Admin*) and do your thing through there.

## Config File


# BIOS Settings
> [!WARNING]
> PC won't be able to boot to macOS if BIOS settings are not set up properly

BIOS settings for ROG Z370-E. *(Refer to OpenCore [Coffee Lake guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#intel-bios-settings))*

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
## Kernel Panic (`Invalid frame pointer`)
> [!NOTE]  
> This really depends on your setup. Refer to OpenCore [Troubleshooting guide](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/kernel-issues.html#kernel-panic-on-invalid-frame-pointer) for a detailed fix.

The [MMIO](https://www.geeksforgeeks.org/memory-mapped-i-o-and-isolated-i-o/) basically maps control registers into the system memory space. As far as I know, it's basically a low-level crash. macOS tries to interact with MMIO regions that it doesn't understand/support, causing the kernel to panic.

You'll know when this happens, because you won't boot and the kernel will show you with these lines:
```
Backtrace terminated-invalid frame pointer
```
```
** In Memory Panic Stackshot Succeeded **
```

### Using `DevirtualiseMmio`
> [!IMPORTANT]
> This issue is very hardware-specific, which is why trial and error is the best way to narrow it down. Make sure you're using OpenCore `Debug` version.

`DevirtualiseMmio` is a quirk that prevents macOS from directly accessing certain MMIO regions. You should probably [read more about it here](https://dortania.github.io/OpenCore-Install-Guide/extras/kaslr-fix.html#finding-the-slide-value), but this is what I did to solve my issue.

1. Enabled `DevirtualiseMmio` in `config.plist` under `Root\Booter\Quirks`
2. Identify bad MMIO regions

	- I identified 6 potentially bad regions, and then converted their values from hex to decimal.
	- Math it out or just use a [converter](https://www.rapidtables.com/convert/number/hex-to-decimal.html).
	- This is my table of values:

| Item # | MMIO Region Address (Hex) | Decimal Value |
|--------|---------------------------|---------------|
| 0      | `0xF800 0000`             | 4,160,749,568 |
| 1      | `0xFE00 0000`             | 4,261,412,564 |
| 2      | `0xFEC0 0000`             | 4,273,995,776 |
| 3      | `0xFED0 0000`             | 4,275,044,352 |
| 4      | `0xFEE0 0000`             | 4,276,092,928 |
| 5      | `0xFF00 0000`             | 4,278,190,080 |

3. Start troubleshooting

	- Don't know which region is bad until...
	- ...block all MMIO except one and try each region (trial and error).
	- In `config.plist`, create new children under `Root\Booter\MmioWhitelist` and make sure each item is a `Dictionary`.
	- Enable each item except one ((`Boolean: False`)).
	- Save, flush, reboot.
	- If it doesn't work, repeat until you find the bad address (no more kernel panic).

## RTC Write Issues
- Prompts boot message *"The system has POSTed in safe mode"* after rebooting from MacOS
- Refer to *[Fixing RTC write issues (guide)](https://dortania.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)* for more info

## Unsynced time after rebooting to W11
One annoying issue is that your clock changes every time you boot to Windows after using macOS. The time is essentially stored on the motherboard, but Windows and macOS interpret this stored time differently.
- Windows does not apply a timezone to the system time
- macOS interprets the system time as [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)

As a result of this, neither will agree with each other and the time displayed on Windows will always be messed up after rebooting from macOS. The way to fix this is to ***change how Windows interprets time*** as UTC.

<details><summary><h3>Quick Fix</h3></summary>

1. Download <file_name.reg> (Right click > Save link as...)
2. Run as Administrator
3. Restart

</details>

<details><summary><h3>Manual fix</h3></summary>

1. Launch the Registry Editor (<kbd>⊞ Win</kbd>+<kbd>R</kbd>, type `regedit`)
2. Go to
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```
3. Right click > New > `DWORD (32-bit)`
4. Name the key `RealTimeIsUniversal` and set the value data to `1`
5. Save and Restart

</details>

<details><summary><h3>Disable fix</h3></summary>

1. Launch `regedit`
2. Go to
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```
3. Delete the key `RealTimeIsUniversal`
4. Restart

</details>

# Other Info
## Compatibility between *Polaris* and *Navi* GPUs (AMD)
Sometime along the project, I upgraded from an **RX 580** to a **RX 5700 XT** because I was getting horrible performance playing *Dragon's Dogma 2*. After some research, I upgraded to a card that should have just been *plug-and-play*. ***Except***, it wasn't. Luckily, the fix was pretty simple (*See [AMD Boot Arguments](https://dortania.github.io/GPU-Buyers-Guide/misc/bootflag.html#amd-boot-arguments)*).

All I did was **added the boot argument** `agdpmod=pikera`, which is required for all [Navi GPUs](https://en.wikipedia.org/wiki/Radeon_RX_5000_series). Pretty sure that means this method should work for all 5000 and 6000 series cards ([*Don't quote me on that*](https://dortania.github.io/OpenCore-Install-Guide/config-HEDT/broadwell-e.html#nvram)).

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
- [x] Complete BIOS settings tables
