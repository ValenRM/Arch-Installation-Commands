# Arch Installation Commands
 The commands displayed below are identical to those used in the [**Arch Install**](https://github.com/ValenRM/ArchInstall) repository.

# Checking EFI Variables
 We must verify that the `efivarfs` kernel module is loaded. Therefore we are going to run the following command:
 ```
 ls /sys/firmware/efi/efivars
 ```
 Ensure that you are booting your system with UEFI enabled if the directory is not found.

 # Establishing an internet connection
 If you are connected via Ethernet, make sure that you can ping external sevices such as **google.com**. If you are using Wi-Fi, you are going to use [**iwd**](https://wiki.archlinux.org/title/iwd).

 - Get into an iteractive prompt using `iwctl`. The interactive prompt will be displayed with a prefix of `[iwd]#`.

 | Command | Description |
 | --- | --- |
 | device list | List all Wi-Fi devices |
 | station **<device>** scan | Initiate a scan for networks (*No output will be displayed*) |
 | station **<device>** get-networks | List all available Wi-Fi networks |
 | station **<device>** connect SSID | Connect to a network |