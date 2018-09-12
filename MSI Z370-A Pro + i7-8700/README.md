# HACKINTOSH 10.13.4 ON MSI Z370-A PRO
This note is intend to install High Sierra 10.13.4 on a MSI Z370-A PRO MotherBoard.
Specs shows as following:

| Type        | Item                      |
| ----------- | ------------------------- |
| CPU         | Intel Coffee Lake i7-8700 |
| Motherboard | MSI Z370-A Pro            |
| Memory      | DDR4 2666 16GB            |
| VideoCard   | Nvidia GTX-1050           |
| PowerSupply | EVGA 80+ 550W             |


## Some success write-ups
Hackintosher Install Guide
https://hackintosher.com/builds/guide-hackintoshing-msi-z370-pro/

Reddit Install Guide
https://www.reddit.com/r/hackintosh/comments/8cpokx/success_i7_8700k_msi_z370apro_gtx1080ti/

macOS-Windows Dual Boot Setup
https://hackintosher.com/guides/hackintosh-dual-boot-windows-10-and-macos-high-sierra/#step5

USB3 Fix
https://www.tonymacx86.com/threads/macos-10-13-4-update.248292/page-9#post-1718100

## BIOS Settings
```
Save & Exit → Restore Defaults : Yes
Settings \ Advanced \ Integrated Peripherals → Network Stack : [Disabled]
Settings \ Advanced \Integrated Peripherals  → Intel Serial IO : [Disabled]
Settings \ Advanced \ Integrated Graphics Configuration → DVMT Pre-Allocated : 128MB+ (Or 64MB if that’s the highest you can go)
Settings \ Advanced \ USB Configuration → XHCI Hand-off : [Enabled]
Settings \ Advanced \ USB Configuration → Legacy USB Support : [Auto]
Settings \ Advanced \ Windows OS Configuration → MSI Fast Boot : [Disabled]
Settings \ Advanced \ Windows OS Configuration → Fast Boot : [Disabled]
Overclocking  → Extreme Memory Profile(X.M.P) : [Enabled]
Overclocking \ CPU Features → Intel Virtualization Tech : [Enabled]
Overclocking \ CPU Features → Intel VT-D Tech : [Disabled]
Settings \ Boot → Boot mode select : [LEGACY+UEFI]
Settings \ Boot → Boot Option #1 : UEFI: “macOS_flash_drive_name“
```
source: [Hackintosher](https://hackintosher.com/builds/guide-hackintoshing-msi-z370-pro/)


## Audio Driver
1. Cleaning the `config.plist`
in `KernelAndKextPatches -> KextsToPatch` remove following dict.
```
10.11-AppleHDA/Realtek ALC...
10.9-10.11-AppleHDA/Realtek ALC1150
AppleHDA/Resources/xml&gt;zml
```
2. Remove Old Kexts (FUCKING IMPORTANT)
Looking for...
```
realtekALC.kext
CloverALC.kext
VoodooHDA.kext
HDA Blocker.kext
HDAEnabler#.kext (I believe the # can be 1, 2, or 3 - but there could be others)
AppleALC.kext (I know we'll eventually be installing this - but we wanna start with a clean slate)
```
On those locations...
```
/Library/Extensions/
/System/Library/Extensions/
/Volumes/EFI/EFI/CLOVER/kexts/10.xx (where 10.xx is all the numbered folders)
/Volumes/EFI/EFI/CLOVER/kexts/Other
```
And **DELETE** THEM

4. Add Audio Inject
in `Devices -> Audio -> Inject` set to 1

3. Injecting Kexts
`SystemParameters -> InjectKexts` set to True
`SystemParameters -> InjectSystemID` set to True

4. Vanillafy `AppleHDA.kext`
You need a vanilla `AppleHDA.kext` to get this work.
**If you don't**, do the following chores.
```shell
sudo kextcache -i / && sudo kextcache -u /

# Get a vanilla AppleHDA.kext and replace it to /S/L/E
# Remove old one
sudo rm -Rf /System/Library/Extensions/AppleHDA.kext

# Then we copy the new kext over
sudo cp -R ~/Desktop/AppleHDA.kext /System/Library/Extensions/AppleHDA.kext

# Fix ownership and permissions
sudo chown -R root:wheel /System/Library/Extensions/AppleHDA.kext
sudo chmod -R 755 /System/Library/Extensions/AppleHDA.kext
```

5. Install the latest `AppleALC.kext`
Download the latest [`AppleALC.kext`](https://github.com/vit9696/AppleALC/releases)
Put it to `/Volumes/EFI/EFI/CLOVER/kexts/Other/`

6. Rebuild Cache
```shell
sudo kextcache -i / && sudo kextcache -u /
```
## Fix HDMI Audio (Not Working)
1. Delete `Lilu.kext` from `/S/L/E` and `/L/E`
2. Rebuild kernel cache
```shell
sudo kextcache -i /
```
3. Put latest `Lilu.kext`(1.2.4) and `AppleALC.kext`(v 1.2.4) to `/EFI/Clover/Kext/Other`
4. Reboot

source: [tonymacx86](https://www.tonymacx86.com/threads/asus-z170-a-lost-hdmi-audio-with-10-13-4.249206/)


## Coffee Lake SSDT (Cannot fix undetected CPU info)
1. check ACPI->SSDT->PluginType in `config.plist`
2. Reboot

source: [tonymacx86](https://www.tonymacx86.com/threads/guide-generate-ssdt-for-coffee-lake-cpu.238311/)

## Aptio Memory Fix (Unconfirmed)
1. Remove `OsxAptioFix2Drv-free2000.efi` and `EmuVariable64UEFI.efi`
2. Drop in `AptioMemoryFix.efi` from the Clover install package
