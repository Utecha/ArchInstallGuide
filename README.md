# Arch Install Process (UEFI)

1. ping archlinux.org
    - If you get successful pings, quit with Ctrl-C

2. cat /sys/firmware/efi/fw_platform_size
    - If you get "64" printed to the Terminal, you are using UEFI

3. lsblk
    - Determine which drive you are going to install Arch onto

4. gdisk /dev/sdX 
    - With X being the drive you want to install to (make sure it's the right one!!!)
    - Enter 'x' to enter Expert command mode
    - Enter 'z' to Zap the drive (aka wipe it)
    - Enter 'y' twice, once on each prompt

5. cgdisk /dev/sdX
    - Here you will make either 3 or 4 partitions (depending on if you prefer a separate /home partition)
    - First first sector, always leave empty! Enter the following sizes for the final sector.
    - Sizes for each sector are up to you. It's recommend no less than 256M for boot. 
    - Swap will vary based on the amount of RAM you have.
    - For root, either use what is left, or recommended around 40GB if you prefer a separate /home partition.
    - For the codes, just hit Enter for both the root and home partitions as 8300 is the default.
      
    ## My Setup (single drive)
    - Size 512M :: Code EF00 :: BOOT
    - Size 16G :: Code 8200 :: SWAP
    - Size 40G :: Code 8300 :: ROOT
    - Size [Rest of Drive] :: Code 8300 :: HOME
    - Hit enter on "Write", type 'yes', then hit enter on "Quit"

6. Confirm changes with lsblk, then format the partitions
    - Note: If you followed this guide, BOOT will always be Partition 1, SWAP 2, ROOT 3, and HOME 4
   
    - BOOT :: mkfs.vfat -F 32 /dev/sdX1
    - SWAP :: mkswap /dev/sdX2 :: swapon /dev/sdX2
    - ROOT :: mkfs.ext4 /dev/sdX3
    - HOME :: mkfs.ext4 /dev/sdX4

7. Mount your filesystems
    - ROOT should always be mounted first!
   
    - ROOT :: mount /dev/sdX3 /mnt
    - BOOT :: mount --mkdir /dev/sdX1 /mnt/boot/efi
    - HOME :: mount --mkdir /dev/sdX4 /mnt/home

8. Generate an updated mirrorlist
    - cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
    - pacman -Sy pacman-contrib
    - rankmirrors -n 10 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist

9. Strap on your seatbelts! Time for the pacstrap!
    - Note: If you want the Linux Zen kernal, replace "linux" with "linux-zen" and "linux-zen-headers"
    - GRUB and efibootmgr are only necessary if you want to use GRUB over systemd boot
    - pacstrap -K /mnt base base-devel linux linux-firmware grub efibootmgr

10. Generate your fstab
    - genfstab -U -p /mnt >> /mnt/etc/fstab
    - *OPTIONAL* Check correct fstab generation :: nano /mnt/etc/fstab

11. Chroot into your install
    - arch-chroot /mnt
    - Note: At this point you will not have a text editor. Install your preferred terminal text editor.
    - Popular options: emacs, nano, neovim, vim, vi
    - pacman -Sy bash-completion
      
    - Generate Locales
        - Open /etc/locale.gen in your favourite text editor
        - Uncomment these lines:
            - en_US.UTF-8 UTF-8
            - en_US ISO-8859-1
        - locale-gen
        - echo LANG=en_US.UTF-8 > /etc/locale.conf
        - export LANG=en_US.UTF-8
          
    - Set up Time Zones
        - To find out which one you need, use ls /usr/share/zoneinfo/
        - If you live in the US and are in Eastern Time, just use the line below.
        - ln -sf /usr/share/zoneinfo/America/New_York > /etc/localtime
          
    - Sync HW Clock
        - hwclock --systohc --localtime
          
    - Generate Hostname
        - echo [YOUR_HOSTNAME] > /etc/hostname
          
    - Enable TRIM support for SSDs (if you have one)
        - systemctl enable fstrim.timer
          
    - Enable 32-bit support (Edit Pacman configuration)
        - nvim /etc/pacman.conf
        - *OPTIONAL* Uncomment "Color", "VerbosePkgLists", "ParallelDownloads"
        - *OPTIONAL* To make the download bars prettier, add ILoveCandy below "ParallelDownloads"
        - Uncomment these lines:
            - [multilib]
            - Include = /etc/pacman.d/mirrorlist
        - pacman -Sy to update packages
          
    - Setup root and user accounts
        - passwd :: Enter this for a root password. Enter your chosen root password.
            - If you do not want a separate root password, skip this step.
        - useradd -m -g users -G wheel,storage,power -s /bin/bash [USERNAME]
        - passwd [USERNAME] :: Enter your chosen password for the user
          
    - Edit sudo configuration
        - EDITOR=nvim visudo
        - Uncomment this line:
            - %wheel ALL=(ALL:ALL)
        - At the EOF, add :: Defaults rootpw (if you set a root password)
          
    - Install Networking services
        - pacman -S networkmanager
        - systemctl enable NetworkManager
          
    - Install graphics drivers
        - For Nvidia
            - pacman -S nvidia-dkms libglvnd nvidia-utils opencl-nvidia lib32-libglvnd lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings
            - nvim /etc/mkinitcpio.conf
            - Add (in this order) inside of Modules=() :: nvidia nvidia_modeset nvidia_uvm nvidia_drm
        - For Intel Integrated Graphics
            - pacman -S mesa lib32-mesa xf86-video-intel vulkan-intel lib32-vulkan-intel
              
    - Install CPU Microcode
        - Intel
            - pacman -S intel-ucode
        - AMD
            - pacman -S amd-ucode
              
    - Install Bootloader of Choice
        - For systemd
            - *TO BE ADDED LATER*
        - For GRUB
            - mount -t efivarfs efivarfs /sys/firmware/efi/efivars/
            - grub-install /dev/sdX
            - grub-mkconfig -o /boot/grub/grub.cfg

12. Finishing Up!
    - exit
    - umount -R /mnt
    - reboot

## Congratulations! You have just installed Arch Linux! :) Btw, you use Arch!
