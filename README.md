# About

These are my notes about the [Seeed Studio Odyssey X86J4105 board](https://wiki.seeedstudio.com/ODYSSEY-X86J4105/).

## Firmware/BIOS/UEFI

See [firmware](firmware/).

## USB keyboards

Be aware that (non)-composite USB keyboards can only be used to enter the BIOS
(by pressing the DELETE key) when Fast Boot is disabled (which is the default
after a BIOS Restore Defaults).

## Poweroff

See https://forum.seeedstudio.com/t/how-to-completely-turn-off-the-network-interfaces-while-odyssey-x86j4105-is-turned-off/255557

### UEFI Boot Loader

Since the board ships with Windows 10 pre-installed, the UEFI bootloader
retains the `Windows Boot Manager` entry as seen by:

```bash
efibootmgr --verbose
```

The command should return the current boot settings, something alike:

```
BootCurrent: 0001
Timeout: 5 seconds
BootOrder: 0001,0002,0000,0005
Boot0000* Windows Boot Manager	HD(1,GPT,93faa71b-80e1-42f4-a7dd-0fb3a1c1a75e,0x800,0x32000)/File(\EFI\MICROSOFT\BOOT\BOOTMGFW.EFI)WINDOWS.........x...B.C.D.O.B.J.E.C.T.=.{.9.d.e.a.8.6.2.c.-.5.c.d.d.-.4.e.7.0.-.a.c.c.1.-.f.3.2.b.3.4.4.d.4.7.9.5.}....................
Boot0001* ubuntu	HD(1,GPT,e2daca90-f32f-4d3e-b696-0244329c6551,0x800,0x100000)/File(\EFI\UBUNTU\SHIMX64.EFI)
Boot0002* UEFI: Built-in EFI Shell	VenMedia(5023b95c-db26-429b-a648-bd47664c8012)..BO
Boot0005* ubuntu	HD(1,GPT,e2daca90-f32f-4d3e-b696-0244329c6551,0x800,0x100000)/File(\EFI\UBUNTU\GRUBX64.EFI)..BO
```

Manually remove the `Boot0000` entry with:

```bash
efibootmgr --verbose --bootnum 0000 --delete-bootnum
```

The command should return the current boot settings, something alike:

```
BootCurrent: 0001
Timeout: 5 seconds
BootOrder: 0001,0002,0005
Boot0001* ubuntu	HD(1,GPT,e2daca90-f32f-4d3e-b696-0244329c6551,0x800,0x100000)/File(\EFI\UBUNTU\SHIMX64.EFI)
Boot0002* UEFI: Built-in EFI Shell	VenMedia(5023b95c-db26-429b-a648-bd47664c8012)..BO
Boot0005* ubuntu	HD(1,GPT,e2daca90-f32f-4d3e-b696-0244329c6551,0x800,0x100000)/File(\EFI\UBUNTU\GRUBX64.EFI)..BO
```

Or better yet, remove all entries, as in my case:

```bash
efibootmgr --verbose --bootnum 0001 --delete-bootnum
efibootmgr --verbose --bootnum 0002 --delete-bootnum
efibootmgr --verbose --bootnum 0005 --delete-bootnum
```

Delete the boot order (`grub-install` will re-create it):

```bash
efibootmgr --verbose --delete-bootorder
```

Then reinstall the `ubuntu` boot entry with:

```bash
grub-install
```

The end result should be:

```
BootCurrent: 0001
Timeout: 5 seconds
BootOrder: 0000
Boot0000* ubuntu	HD(1,GPT,e2daca90-f32f-4d3e-b696-0244329c6551,0x800,0x100000)/File(\EFI\ubuntu\shimx64.efi)
```

Then reboot the machine:

```bash
reboot
```

After reboot, the boot settings should be something alike:

```
BootCurrent: 0000
Timeout: 5 seconds
BootOrder: 0000,0001,0002
Boot0000* ubuntu	HD(1,GPT,e2daca90-f32f-4d3e-b696-0244329c6551,0x800,0x100000)/File(\EFI\UBUNTU\SHIMX64.EFI)
Boot0001* ubuntu	HD(1,GPT,e2daca90-f32f-4d3e-b696-0244329c6551,0x800,0x100000)/File(\EFI\UBUNTU\GRUBX64.EFI)..BO
Boot0002* UEFI: Built-in EFI Shell	VenMedia(5023b95c-db26-429b-a648-bd47664c8012)..BO
```

**NB** The entries after `Boot0000` are automatically populated by the UEFI.

### UEFI Reboot To Firmware

The system can reboot to the firmware using:

```bash
systemctl reboot --firmware-setup
```

For more details see:

* https://www.freedesktop.org/software/systemd/man/systemctl.html#--firmware-setup
* [efi_reboot_to_firmware_supported](https://github.com/systemd/systemd/blob/e7a8f6b66fa3efd298120ef5d6c972d7b4df51c7/src/shared/efi-loader.c#L70-L103)
  * [EFI_OS_INDICATIONS_BOOT_TO_FW_UI](https://github.com/systemd/systemd/blob/e7a8f6b66fa3efd298120ef5d6c972d7b4df51c7/src/shared/efi-loader.c#L31)
* [get_os_indications](https://github.com/systemd/systemd/blob/e7a8f6b66fa3efd298120ef5d6c972d7b4df51c7/src/shared/efi-loader.c#L105-L158)

SystemD will manipulate these UEFI variables:

```bash
efivar --name 8be4df61-93ca-11d2-aa0d-00e098032b8c-OsIndicationsSupported --print
efivar --name 8be4df61-93ca-11d2-aa0d-00e098032b8c-OsIndications --print
```

```
GUID: 8be4df61-93ca-11d2-aa0d-00e098032b8c
Name: "OsIndicationsSupported"
Attributes:
	Boot Service Access
	Runtime Service Access
Value:
00000000  03 00 00 00 00 00 00 00                           |........        |
```

```
GUID: 8be4df61-93ca-11d2-aa0d-00e098032b8c
Name: "OsIndications"
Attributes:
	Non-Volatile
	Boot Service Access
	Runtime Service Access
Value:
00000000  00 00 00 00 00 00 00 00                           |........        |
```

## Ubuntu

Ubuntu 20.10 works out of the box.

## WiFi

The Wifi/Bluetooth module has two IPEX MHF4 RF antenna connectors.
