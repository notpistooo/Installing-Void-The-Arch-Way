# Installing Voidlinux the Archlinux way

## NOTE:
this repo is inspired by Jake@Linux \
his tutorial: https://jpedmedia.com/tutorials/void_install/index.html \
his youtube video: https://www.youtube.com/watch?v=63f3EWZ-56I


## Introduction
<p>
This is a (not complete) guide to installing voidlinux using purely commands or as i like to say the archlinux way,<br>
This is not an official guide, it just a way for me to pass time.<br>
I'm a NEW voidlinux user, any mistakes would be appreciated to be mentionned.<br>
I would do my best to follow the Archlinux wiki.<br>
I am going to be listing the way to installing void on hardware (Not a VM), idk if there is any big differences.<br>
</p>

## TODO:
- 1.2 Verify Signature.
- make sure if this guide works with other $ARCH
- wired connection.
- Better English.
- Better Explanation.
- Post Install Recommendation.
- Jumping stuff...

## 1.Pre-Installation:
### 1.1 Aquiring the installation image:
Visit the <a href="https://voidlinux.org/download/">Download</a> page,
this guide would assume that you are using base image x86_64 Architecture (glibc),
If you have downloaded the xfce version just make sure to run these commands with root privleges.

### 1.3 Prepare an installation medium:
You can use a tool like rufus, ventoy, or any kind of tool to "burn" the .iso to a usb disk,
or you can burn it into a cd if you want.

### 1.4 Boot the live environment:
Depending on your pc/laptop, press {Esc, F9, F12, ...} to bring the boot menu and boot from installation medium you setup,
if you have the boot menu disabled you can change the boot order from the BIOS.
once you boot into void, log in using root as username, voidlinux as password.
start bash shell using the command `PS1='[ \W ] # ' /bin/bash`

NOTE: in some cases the arch medium would mount itself to the RAM, this is not the case for void, KEEP THE INSTALLATION MEDIUM PLUGGED,

### 1.5 Set the console keyboard layout and font:
To change the keyboard layout, run the command `loadkeys "*Layout*"` \
all layout can be found in the `/usr/share/kbd/keymaps` directory

to change the font run the command `setfont "*FONT*"` \
to change back to default font run `setfont` \
to make the font bigger run the command `setfont -d`, this will double the size of the font. \
all terminal fonts can be located in the `/usr/share/kbd/consolefonts/` directory

### 1.6 Verify the boot mode:
To verify the boot mode run the command `cat /sys/firmware/efi/fw_platform_size`, \
if it returns 64 => booted in UEFI mode and has a 64-bit x64 UEFI.
if it returns 32 => booted in UEFI mode and has a 32-bit IA32 UEFI.
If the file does not exist => BIOS/ or CSM.

### 1.7 Connect to the internet:
for wireless: i just run the `void-installer` and i setup my network and then abort the installation.
verify using `ping voidlinux.org`

### 1.8 Partition the disks:
to list all the disks run `lsblk`
there is alot of guides on how to partition the disk, for my case i will make<br>
EFI GPT:
- /dev/sda1: ROOT (220G) Linux filesystem
- /dev/sda2: EFI (583M) EFI System
- /dev/sda3: SWAP (3G) Linux swap<br>

BIOS MBR:
- /dev/sda1: ROOT (220G) Linux
- /dev/sda2: BOOT (583M) Linux
- /dev/sda3: SWAP (3G) Linux swap / Solaris 

### 1.10 Format the partitions:
you can use anyfile system for the root partition you want, i will use ext4.
- ROOT `mkfs.ext4 /dev/sda1` 
- EFI/BOOT `mkfs.vfat /dev/sda2` 
- SWAP `mkswap /dev/sda3`

### 1.11 Mount the partitions:
- ROOT `mount /dev/sda1 /mnt` 
- EFI/BOOT `mount /dev/sda2 /mnt/boot --mkdir` 
- SWAP `swapon /dev/sda3` \
to check if they mounted correctly run `lsblk`

## 2.Installation
### 2.1 Choose mirrors:
For a list with all mirrors visit <a href="https://xmirror.voidlinux.org/">Mirrors</a> page
i will go with the Tier 1 Globally available: `https://repo-fastly.voidlinux.org/` \
add the following to the link:
- glibc: /current
- musl: /current/musl \
run the following commands: \
`REPO=https://repo-fastly.voidlinux.org/current` \
`ARCH=x86_64`

### 2.2 Install essential packages:
run the command:
`XBPS_ARCH=$ARCH xbps-install -S -R "$REPO" -r /mnt base-system linux-mainline neovim` \
you don't have to use neovim as your text editor,
you can use vim or nano or any other tool.
it might take a while, grab a coffee or Shaii Tea~

## 3.Configure the system:
### 3.1 Fstab:
find out the UUID of your partitions: \
`ROOT_UUID=$(blkid -s UUID -o value /dev/sda1)` \
`EFI_UUID=$(blkid -s UUID -o value /dev/sda2)` or `BOOT_UUID=$(blkid -s UUID -o value /dev/sda2)` \
`SWAP_UUID=$(blkid -s UUID -o value /dev/sda3)` 

write the fstab: \
`cat << EOF > /mnt/etc/fstab` \
`# /dev/sda1` \
`UUID=$ROOT_UUID  /  ext4  rw,relatime  0 1` \
`# /dev/sda2` \
`UUID=$EFI_UUID  /boot  vfat  defaults,noatime  0 2` or `UUID=$BOOT_UUID  /boot  vfat  defaults,noatime  0 2` \
`# /dev/sda3` \
`UUID=$SWAP_UUID  none  swap  defaults  0 0`

### 3.2 Chroot:
to chroot into the newly setup envirenment run the commands: \
`for dir in dev proc sys run; do mount --rbind /$dir /mnt/$dir; mount --make-rslave /mnt/$dir; done` \
`cp /etc/resolve.conf /mnt/etc/` \
`PS1='(chroot) # ' chroot /mnt/ /bin/bash`

### 3.3 Time:
you can list out all available timezones by running the following command:
`ls /usr/share/zoneinfo` \
to setup your time run the command \
`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`

### 3.4 Localization:
Edit the `/etc/default/libc-locales` and uncomment your proper locales and then run the command to reconfigure \
`xbps-reconfigure -f glibc-locales`

### 3.5 Network configuration:
Create the hostname file:
`echo "<$HOSTNAME>" > /etc/hostname` \

Add the following lines to `/etc/hosts` \
`127.0.0.1        localhost` \
`::1              localhost` \
`127.0.1.1        <$HOSTNAME>.localdomain        <$HOSTNAME>`

### 3.7 Root password
Set the root password
`passwd`

### Other:
#### Add a user:
`useradd -mg users -G wheel -s /bin/bash <$USERNAME>`

#### Set the user password:
`passwd <$USERNAME>`

#### change the root shell
`chsh -s /bin/bash root`

#### give the user root privileges
uncomment the line `%wheel ALL=(ALL:ALL) ALL` from visudo, run the command: \
`EDITOR=nvim visudo`

#### Sync your repo:
`xbps-install -S` 

This step is not required unless you want to be able to access software that does not have free licenses or if you are a gamer or need 32bit packages \
`xbps-install void-repo-nonfree` \
`xbps-install -S` \
`xbps-install void-repo-multilib` \
`xbps-install -S`

#### NOTE: You can edit your fstab `/etc/fstab` if you made a typo mistake earlier.

### 3.8 Boot loader
We would be installing grub \
BIOS: \
`xbps-install -S grub` \
`grub-install --bootloader-id="Void" /dev/sda` \
NOTE: this command usuelly works, no other options should be passed to the command \
`grub-mkconfig -o /boot/grub/grub.cfg` 

UEFI: \
`xbps-install -S grub-x86_64-efi` \
`grub-install --efi-directory=/boot --bootloader-id="Void" /dev/sda` \
`grub-mkconfig -o /boot/grub/grub.cfg`

## 4.Finishing touches:
install any programs you want, 
if you are using wireless connection install NetworkManager \
`xbps-install -S NetworkManager` \
`ln -s /etc/sv/NetworkManager /var/service` \
`ln -s /etc/sv/dbus /var/service` 
#### NOTE: when using NetworkManager run it as sudo `sudo nmtui`


## 5.Reconfigure programs
run the following command to make sure all packages are configured correctly: \
`xbps-reconfigure -fa`

## 6.End
Exit Chroot: \
`exit`

Unmount Partitions: \
`swapoff /dev/sda3` \
`umount -R /mnt`

Reboot system: \
`reboot`
