# OC-Hackintosh-Z370-E
OpenCore EFI for a machine running i5-8600K, RX 580, on an ROG Z370-E Board

# Hardware Specifications
- **Motherboard**: [ROG STRIX Z370-E GAMING](https://rog.asus.com/motherboards/rog-strix/rog-strix-z370-e-gaming-model/)
	- **Ethernet**: Intel I219-V (2)
- **CPU**: Intel(R) Core(TM) i5-8600K CPU @ 3.60GHz
- **GPU**: AMD Radeon RX 580
- **Memory**: A-DATA XPG DDR4 2400MHz (4x8GB)
- **Storage**: Crucial MX500 500GB 3D NAND SATA SSD

# OS Settings
- **SMBIOS**: iMac19,1
- **MacOS Version**: Ventura 13.6.3

# Checklist
### 
- [ ] Fix Sleep
- [ ] [Add GUI and Boot-chime](https://dortania.github.io/OpenCore-Post-Install/cosmetic/gui.html)
- [ ] Write Issues section of README
- [ ] Finish README

# BIOS Settings
BIOS settings for ROG Z370-E. *(Refer to OpenCore [Coffee Lake guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#intel-bios-settings))*
## Disable:
- *Fast Boot*
	- Boot\Fast Boot
- *Secure Boot*
	- Boot\Secure Boot
- *Serial/COM Port*
	- Advanced\Onboard Devices\Configuration
- *Parallel Port*
	- N/A
- *VT-d*
	- Advanced\System Agent (SA) Configuration
- *CSM*
	- Boot\CSM
- *Thunderbolt*
	- N/A
- *Intel SGX*
	- Advanced\CPU Configuration
- *Intel Platform Trust*
	- Advanced\PCH-FW Configuration
		- TPM: Discrete
- *CFG Lock*
	- Advanced\CPU Configuration\CPU-Power Management Control

## Enable:
- VT-x
	- Advabced\CPU Configuration
		- Intel Virtualization Technology
- Above 4G Decoding

# Features
## Working:
- Ethernet
- All USB Ports
- Time Syncing [*MacOS ↔ W11*]

## Not Working:
- Wi-Fi/Bluetooth

# Issues:
## Kernel Panic (`Invalid frame pointer`)

## RTC Write Issues
- Prompts boot message *"The system has POSTed in safe mode"* after rebooting from MacOS
- Refer to *[Fixing RTC write issues (guide)](https://dortania.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)* for more info
