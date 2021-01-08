# Lenovo ThinkPad T530 Hackinosh OpenCore (DSTD-less)

This Repo contains an EFI Folder with configs for running macOS Catalina or Big Sur with either a patched DSDT or DSDT-less on the Lenovo T530 Laptop. The EFI for running macOS on the Lenovo T530 includes 2 configs:

1. config_DSDT.plist

This config is working 100% for T530 Models wih both HD (AAPL,ig-platform-id 03006601) or HD+ Display (AAPL,ig-platform-id 04006601). If you just want to have
a well running System, use this! You need to rename the config to config.plist in order to boot with this. But before you do, open the config and have a look at the "ACPI > Add" section. Enable either DSDT-HD.aml or DSDT-HD+.aml (never both) depending on the display panel of your T530. Check the comments of the entries to decide which one you need to enable. By default, the DSDT for HD+ panels is enabled.

NOT WORKING:

- You can't boot Windows from OpenCore's BootPicker if you use a single HDD/SSD for both Windows and MacOS. It gives you ACPI Errors. Workaround: use the F12 Bootmenu and select "WindowsBootManager" instead to bypass OpenCore and boot windows (which is recommended anyway). Otherwise use DSDT-less config instead

2. config_DSDT-less.plist

This config is for running macOS without a patched DSDT – it relies on ACPI Hotpatches instead (SSDTs and ACPI patches in the config) which is the recommended method for OpenCore. Since this method does not rely on having a patched DSDT which might mismatch the System's DSDT of the installed BIOS Version, the process of hotpatching is more precise and independent of the installed BIOS. Instead of just replacing the whole system DSDT with the patched one during boot, only the things which need patching are patched. This makes the system boot a bit faster, runs smoother and snappier. 

The default config is for T530 Models with HD+ displays (≥1600x900 px). If you have a model with a HD panel you need to add the correct Framebuffer-Patch for IntelHD 4000 (AAPL,ig-platform-id 03006601).

NOT WORKING:

- Lid: Sleep/Clamshell Mode and switching over the Main Display to an External Monitor when the lid is closed
- Power LED keeps pulsing after exiting sleep

Any help on getting the lid sleep fixed is highly appreciated!

INCOMPATIBLE COMPONENTS:

- Intel Bluetooth/WIFI. You need a compatible card and a BIOS Unlock to disable the WLAN Card Whitelist using 1vyrain
- Discrete NVIDIA GPU – model not supported by macOS. Must be disabled in BIOS!
- Fingerprint Sensor - model not supported by macOS
- VGA Port is not working. More info here: https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#vga-support

## PRE-INSTALL PREPARATIONS: DO's and DONT's

Before copying the EFI onto your SSD/HDD, you should check the following:

- Test the EFI using a FAT32 formatted USB Stick first
- Copy over your existing SMBIOS Infos of create a new serial, MLB, etc. using GenSMBIOS (Catalina requires MacBookPro10,X; Big Sur needs MacBookPro11,X) and copy the Information to PlatformInfo > Generic.
- The SSDT-PM.aml inside the ACPI Folder is for an i7 3630QM Processor. If you have a differnt CPU, disable it and create your own using ssdtPRGEN in Postinstall.
- If you use a different CPU enable the 2 Patches under "ACPI > delete" and save the config, so that the CPU runs full speed.
- Wifi/Bluetooth:
    - Built-in Intel Wifi/Bluetooth Cards don't work. But you can have a look at OpenIntelWireless Kext: https://github.com/OpenIntelWireless/itlwm
    - 3rd Party cards require 1vyrain jailbreak to unlock the BIOS in order to disable WLAN Whitelist (unless the 3rd party card is whitelisted)
    - Broadcom Cards require an additional kext for Bluetooth. Either BrcmFirmwareData.kext in "EFI > OC > Kexts" which will be injected through OpenCore or
      BrcmFirmwareRepo.kext which needs to installed into S/L/E since it cannot be inject by bootloaders, but works a bit more efficient.
    - If you use a different vendor than Broadcom for Bluetooth/Wifi you should replace the Kext(s) for networking for your device and update your config.
- If you create Snapshots for the DSDT-less config using ProperTree, make sure to disable the "ACPI > Add" entries for DSDT files afterwards. Best practice would be to delete both DSDTs from the EFI anyway, if you use the DSDT-less config.
- DON'T create Snapshots for the config_DSDT.plist which is using the DSDT Files. Because this will add all the SSDTs back in, which is unnecessary since all these changes are defined in the patched DSDT already. If you plan to use the DSDT-based config, you might as well delete all of the SSDTs except for SSDT-PM.
- DON'T Update VoodooPS2Controller.kext! The current version doesn't work well with the Trackpad even with an additional Trackpad SSDT. So exclude it from updates.

## INSTALLATION

0. Download EFI Folder from the "Releases Section" on the right and unpack it
1. Read "Preparations" Section first
2. Rename the config file of your choice to "config.plist"
3. Mount the EFI
4. Replace EFI Folder
5. Restart
6. IMPORANT: Perform an NVRAM Reset (in Bootpicker, hit Space Bar and select Clean NVRAM). Especially important when switching from a DSDT to DSDT-less config!
7. Reboot again
8. Select macOS to boot. It's currently configured for running Catalina. If you want to run Big Sur, you need to use SMBIOS 11,x. You can research a suitable/matching SMBIOS for your CPU on everymac.com

## POST-INSTALL

- Fixing CPU Power Management (only necessarry if you use a differnt CPU than i7 3630QM).

	1. Open Config
	2. Enable the 2 Patches under "ACPI > Delete" (Drop CpuPm and Drop Cpu0Ist)
	3. Save config and reboot
	3. Install ssdtPRGen using terminal: https://github.com/Piker-Alpha/ssdtPRGen.sh
	4. Open Terminal and type: sudo /Users/YOURUSERNAME/ssdtPRGen.sh
	5. Go to Users/YOURUSERNAME/Library/ssdtPRGen. There you'll find an ssdt.aml
	6. Rename ssdt.aml to SSDT-PM and replace the one in EFI > OC > ACPI with it
	7. Disable the two patches from step 2 again.
	8. Save config and reboot.
	
NOTE: You can also add modifiers to the terminal command for building the SSDT. You can - for example - drop the low frequency from their default 1200 MHz to 900 MHz in 100 mHz increments, but no lower than that. Otherwise the System Crashes during boot. I suggests you experiement a bit.

- Fixing Sleep: If you have issues with sleep, run the following commands in Terminal:

	sudo pmset hibernatemode 0
	sudo rm /var/vm/sleepimage
	sudo touch /var/vm/sleepimage
	sudo chflags uchg /var/vm/sleepimage

- Switch Command and Option Keys. By default, the ALT key is the CMD Key in macOS and the Windows Key is the Option Key. To switch them around open System Settings > Keyboard. On the right there's a button for Special Keys. Just switch the Option and Command keys to the opposite and everything's fine.
	

## BIOS SETTINGS

- CONFIG [TAB]
	- USB
		- USB UEFI BIOS Support: Enabled
        - USB 3.0 Mode: Enabled
    - Display
        - Boot Display Device: ThinkPad LCD
        - OS Detection for NVIDIA Optimus: Disabled (if your T530 deosn't have a discrete GPU you don't see this Option)
    - Serial ATA (SATA)
        - SATA Controller Mode: XHCI
	-CPU
		- Core Multi-Processing: Enabled
		- Intel (R) Hyper-Threading: Enaybled (CPU must support it)

- SECURITY [TAB]
	- Security Chip: Disabled
	- UEFI BIOS Update Options
		- Flash BIOS Updating by End-Users: Enabled
		- Secure Rollback Prevention: Enabled
	- Memory Protection: Enabled
	- Virtualization
		- Intel (R) Virtualization Technology: Enabled (Relevant for Windows only, disabled for macOS via config)
	- I/O Port Access (Disable the following devices/features)
		- Wireless WAN
		- ExpressCard Slot
		- eSATA Port
		- Fingerprint Reader
		- Antitheft
			- Current Setting: Disabled
			- Computrace: Disabled
		- Secure Boot
			- Secure Boot: Disabled

- STARTUP [TAB]
	- BOOT (Set the Order of Boot devices. Set HDD/SSD as firs device)
	- UEFI/Legacy Boot: UEFI only
		- CSM Support: Disabled
	- Boot Mode: Quick
	- Boot Order Lock: Enabled. Enable this after you've set-up the order of the Boot Drives. This prohibits WindowsBootManager from taking over the first slot of the boot drives.
		
## CREDITS and THANK YOUs:

- n4ru for 1vyrain jailbreak to remove WLAN whitelist: https://github.com/n4ru/1vyrain
- Acidanthera and Team for the OpenCore Bootloader: https://github.com/acidanthera/OpenCorePkg
- Dortantia for the OpenCore Install Guide: https://dortania.github.io/OpenCore-Install-Guide
- Corpnewt for incredibly useful Tools like SSDTTime, GenSMBIOS and ProperTree: https://github.com/corpnewt
- Piker-Alpha for ssdtPRGen: https://github.com/Piker-Alpha/ssdtPRGen.sh
- Al6042 and Sascha_77 from Hackintosh-Forum.de for providing patched DSDTs and initial EFI Folder for the T530
- Daliansky for OC Little Repo with all the ACPI Hotpatches for OpenCore: https://ooh3dpsdytm34sfhws63yjfbwy--github-com.translate.goog/daliansky/OC-little
- Real Kiro for Clover EFI with ACPI Patches for referencing: https://translate.google.com/translate?sl=auto&tl=en&u=https://github.com/RealKiro/Hackintosh
- Rehabman for all USBInjectall.kext, Laptop and DSDT patchig guides: https://github.com/RehabMan
