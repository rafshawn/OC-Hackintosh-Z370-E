# Issues
## Kernel Panic (`Invalid frame pointer`)
> [!NOTE]
> This really depends on your setup.[^1] Refer to OpenCore [Troubleshooting guide](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/kernel-issues.html#kernel-panic-on-invalid-frame-pointer) for a detailed fix.

The [MMIO](https://www.geeksforgeeks.org/memory-mapped-i-o-and-isolated-i-o/) basically maps control registers into the system memory space. As far as I know, it's basically a low-level crash. macOS tries to interact with MMIO regions that it doesn't understand/support, causing the kernel to panic.

You'll know when this happens, because you won't boot and the kernel will show you with these lines:
```
Backtrace terminated-invalid frame pointer
```
```
** In Memory Panic Stackshot Succeeded **
```

[^1]: I stopped experiencing this issue after upgrading my GPU. I know because my `MmioWhitelist` is empty and macOS boots just fine.

### Using `DevirtualiseMmio`
> [!IMPORTANT]
> This issue is very hardware-specific, which is why trial and error is the best way to narrow it down. Make sure you're using OpenCore `Debug` version.

`DevirtualiseMmio` is a quirk that prevents macOS from directly accessing certain MMIO regions. You should probably [read more about it here](https://dortania.github.io/OpenCore-Install-Guide/extras/kaslr-fix.htmll#using-devirtualisemmio), but this is what I did to solve my issue.

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
	- Enable each item except one (`Boolean: False`).
	- Save, flush, reboot.
	- If it doesn't work, repeat until you find the bad address (no more kernel panic).

## RTC Write Issues
- Prompts boot message *"The system has POSTed in safe mode"* after rebooting from MacOS
- Fix was simple:
	1. Open `config.plist` and navigate to
	 ```
	 Root\Kernel\Quirks
	 ```
	2. `DisableRtcChecksum` → `True`
	3. Save, flush, reboot to test
- Refer to *[Fixing RTC write issues (guide)](https://dortania.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)* for more info

## Unsynced time after rebooting to W11
One annoying issue is that your clock changes every time you boot to Windows after using macOS. The time is essentially stored on the motherboard, but Windows and macOS interpret this stored time differently.
- Windows does not apply a timezone to the system time
- macOS interprets the system time as [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)

As a result of this, neither will agree with each other and the time displayed on Windows will always be messed up after rebooting from macOS. The way to fix this is to ***change how Windows interprets time*** as UTC.

### Quick Fix
1. [Download `timesync_fix.reg`](.\Issues\timesync_fix.reg) (Right click > Save link as...)
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
4. Restart