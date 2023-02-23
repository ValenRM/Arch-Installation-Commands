# Arch Installation Commands
 The commands displayed below are identical to those used in the [**Arch Install**](https://github.com/ValenRM/ArchInstall) repository.

# Checking EFI Variables
 We must verify that the `efivarfs` kernel module is loaded. Therefore we are going to run the following command:
 ```
 ls /sys/firmware/efi/efivars
 ```
 Ensure that you are booting your system with UEFI enabled if the directory is not found.

 # Establishing an Internet Connection
 If you are connected via Ethernet, make sure that you can ping external sevices such as **google.com**. If you are using Wi-Fi, you are going to use [**iwd**](https://wiki.archlinux.org/title/iwd).

 - Get into an iteractive prompt using `iwctl`. The interactive prompt will be displayed with a prefix of `[iwd]#`.

 | Command | Description |
 | :--- | :--- |
 | device list | List all Wi-Fi devices |
 | station ***device*** scan | Initiate a scan for networks (*No output will be displayed*) |
 | station ***device*** get-networks | List all available Wi-Fi networks |
 | station ***device*** connect SSID | Connect to a network |

 # Creating Disk Partitions
 To check the current disks and partitions, use the following command:
 ```
 fdisk -l
 ```
 We will create three partitions
 - EFI
 - Swap
 - Root

 Launch the [**fdisk**](https://wiki.archlinux.org/title/fdisk) command-line utility by entering the following command:
 ```
 fdisk /dev/<DISK_NAME>
 ```

 ## Creating EFI Partition

 | Command | Description
 | :---: | :--- |
 | g | Creates a new GPT partition table
 | n | Creates a new partition
 | 1 | Partition Number
 |  | Leave Empty
 | +550M | EFI Partition Size

 ## Creating Swap Partition

 | Command | Description
 | :---: | :--- |
 | n | Creates a new partition
 | 2 | Partition Number
 |  | Leave Empty
 | +4G | Swap Partition Size

 ## Creating Root Partition

 | Command | Description
 | :---: | :--- |
 | n | Creates a new partition
 | 3 | Partition Number
 |  | Leave Empty
 | +100G | Root Partition Size (Leave empty for the max capacity)

 ## Defining Partition Types
 We are now going to specify the EFI partition type for partition one.

 | Command | Description
 | :---: | :--- |
 | t | Change a partition type
 | 1 | Partition number
 | 1 | EFI partition type

 We also need to set partition two as a Swap partition

 | Command | Description
 | :---: | :--- |
 | t | Change a partition type
 | 2 | Partition number
 | 19 | Swap partition type

 ## Writing Changes

 | Command | Description
 | :---: | :--- |
 | w | Write changes

 ## Formatting Partitions
 We will format all three partitions.
 ### Formatting EFI Partition
 ```
 mkfs.fat -F 32 /dev/<efi-partition>
 ```

 ### Formatting Swap Partition
 ```
 mkswap /dev/<swap-partition>

 swapon /dev/<swap-partition>
 ```
### Formatting Root Partition
```
mkfs.ext4 /dev/<root-partition>
```

# Mounting the File System
