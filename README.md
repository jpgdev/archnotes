ARCH x64 installation
=====================

## Partitions recommendations

| Partition | Disk type | Size | Comments|
| ---   | ---  | ---    | --- |
| /     | SDD  | 15 GB  | ROOT |
| /boot | SSD? | 200 MB | ...|
| /home | HDD  | 40 GB  | ...  |
| /var  | HDD  | 20 GB   | App, variables, data |
| swap  | N/A  | N/A    | **Not needed for 8gb ram** |

## How to basic install

ISO used: `archlinux-2014.12.01-dual.iso`


### Basic Installation

|  Specs used on the VM   | |
| --- | -----    |
| HDD | 35 GB    |
| RAM | 2 GB     |
| CPU | 4 cores  |

#### Followed this video (Video)

[![http://www.youtube.com/watch?v=Wqh9AQt3nho)](http://img.youtube.com/vi/Wqh9AQt3nho/0.jpg)](http://www.youtube.com/watch?v=Wqh9AQt3nho)

#### Dual boot Windows 8 & Arch (Video)

[![http://www.youtube.com/watch?v=METZCp_JCec)](http://img.youtube.com/vi/METZCp_JCec/0.jpg)](http://www.youtube.com/watch?v=METZCp_JCec)

### Steps used in the video:

1.  Check what disks we have : `fdisk -l` or `lsblk -f`
2.  Open disk partition utility : `cfdisk /dev/sda` (MBR) OR `cgdisk /dev/sda` (GPT) (**/dev/sda** = disk name)
    -  If asked use DOS as label (It's actually MBR)
4.  Create a swap partition with half the ram as size (/dev/sda1 in this
    case)
    -   **The partition should be Primary**
    -   Click `[TYPE]` and set it to `Linux Swap` (Number 82)

5.  Create another partition for the rest (/dev/sda2 in this case)
    -   **The partition should be Primary**
    -   Set the Bootable Flag to TRUE

6.  Click `[WRITE]`
7.  Click `[QUIT]`
8.  Now we format the partition with the EXT4 file system

    -   `mkfs.ext4 /dev/sda2` (**/dev/sda2** being the bootable
        partition)

9.  Mount the root partition to /mnt
    -   `mount /dev/sda2 /mnt`

10. Mount other partitions (may need to `mkdir /mnt/home` before)
    -  `mount /dev/sda3 /mnt/home`
    -  `mount /dev/sda4 /mnt/var`

10. Make a SWAP file on SWAP partition
    -   `mkswap /dev/sda1`

11. Utilize the SWAP
    -   `swapon /dev/sda1`

12. Setup the network connection
    -   Wired connection : Should already be connected
    -   Wifi connection : `wifi-menu` (google how to use it)

13. Install base system + development libraries (in case some package
    need it as a dependancy) to root drive.
    -   Use pacman: `pacstrap /mnt base base-devel`

14. If the EFI Partiton DIrectory was created by another OS like windows.

    -  `mkdir -p /mnt/boot/efi` then mount the EFI Partition Directory `mount /dev/sda2 /mnt/boot/efi` (**/dev/sda2** is the vfat partition containing the EFI Partition Directory)

14. The base system should now be installed on the root partition.

15. Give ourself full access to the system
    -   `arch-chroot /mnt`

16. Create new root password
    -   `passwd` and then enter the password

17. Edit locale
    -   `nano /etc/locale.gen`
    -   Find the right language, starts with **en\_CA** or **en\_US** in my case. Uncomment the corresponding `'...UTF-8 UTF-8'` line and `'...ISO-8859-1'` line.
    -   `CTRL + O` to write line + ENTER
    -   `CTRL + X` to exit

18. Generate LOCALE
    -   `locale-gen`

19. Specify a TimeZone

    -   `cd /usr/share/zoneinfo`
    -   `ls`
    -   `cd /usr/share/zoneinfo/Canada`
    -   `ls`
    -   `ln -s /usr/share/zoneinfo/Canada/Eastern /etc/localtime`

20. Set hostname for the system
    -   `echo NAME > /etc/hostname`
    -   The NAME is the machine name (ex. arch-laptop)

21. Download GRUB (or another bootloader)
    -   `pacman -S grub`

22. Install GRUB to hard drive

    - For Windows Dual Boot
        - Mount the EFI directory partition to `/mnt/boot` (should be the **vfat** partition)
        - Then use `grub-install --efi-directory=/boot/efi --target=x86_64-efi /dev/sda`

    - For basic installation
        -  `grub-install /dev/sda`
    -   NOTE: Use the hard drive name, **NOT** the partition. (here:
        /dev/sda).

23. Generate init file that GRUB will use to load linux.
    -   `mkinitcpio -p linux`

24. Create GRUB config file
  - If you have multiple OSes, install `os-prober` before running `grub-mkconfig`(?) .
  - **If you have error with os-prober** and it can't find the other OSes, you have to add it manually [related article from the Arch Wiki on GRUB](https://wiki.archlinux.org/index.php/GRUB#Windows_installed_in_UEFI-GPT_Mode_menu_entry).
      - You have to add the menuentry manually in the  `/etc/grub.d/40_custom`, replace the `$hints_string` and `$fs_uuid` with their values as shown in the wiki.
     	- Example:

			```bash
			if [ "${grub_platform}" == "efi" ]; then
				menuentry "Microsoft Windows Vista/7/8/8.1 UEFI-GPT" {
					insmod part_gpt
					insmod fat
					insmod search_fs_uuid
					insmod chain
					search --fs-uuid --set=root $hints_string $fs_uuid
					chainloader /EFI/Microsoft/Boot/bootmgfw.efi
			}
			fi
			```

      - To find the `$hints_string` : `grub-probe --target=hints_string $esp/EFI/Microsoft/Boot/bootmgfw.efi` (the `$esp` can be replaced by the path to the /EFI, normally `/boot/efi`)
      - To find the `$fs_uuid` : `grub-probe --target=fs_uuid $esp/EFI/Microsoft/Boot/bootmgfw.efi` (the `$esp` can be replaced by the path to the /EFI, normally `/boot/efi`)
    -  Then run  `grub-mkconfig -o /boot/grub/grub.cfg` to create the GRUB config file.

25. Exit the chroot session
    -   `exit`

26. Generate an fstab file based on our HD
    -   An **FSTAB** file contains informations on the partitions on the
        system.
    -   `genfstab -p /mnt > /mnt/etc/fstab`
    -   Might need to modify the file for SSD partition as seen
        [here](https://www.youtube.com/watch?v=kQFzVG4wZEg) around
        14:30.

27. Unmount the root device (and others if applicable)
    -   `umount -R /mnt`

28. Reboot the system
    -   `reboot`

29. **The OS is now installed.**

Setup user
----------

[Guide on what does the parameters do](http://www.tecmint.com/add-users-in-linux/)

1.   Create user

    -   `useradd -m -g users jp`
    -   `passwd jp` (then type password)

2.   Set the user as sudoer

    -   `nano /etc/sudoers`
        -   find the line : `root ALL=(ALL) ALL`
        -   write under: `jp ALL=(ALL) ALL`

Setup the Network interface
---------------------------

#### Enable DCHP upon boot

1.   Find network interface

2.   List network interfaces : `ls /sys/class/net`

    -   **Note** : A wired connection should start with **en**.

3.   Enable the interface

    -  `systemctl enable dhcpcd@eno16777736`
    -   **Note** : `eno16777736` is the interface name for this instance.
    -   **Note** : to **disable**
        -   `systemctl disable dhcpcd@eno16777736`

4.   Start the interface

    -   `systemctl start dhcpcd@eno16777736`
    -   **Note** : `eno16777736` is the interface name for this instance.

SSH
---

- Enable SSH Daemon : `sudo systemctl enable sshd.service` & `sudo systemctl enable sshd.service`
- Config file : `/etc/ssh/sshd_config`
		- Enable Password authentification (instead of Public/Private Key) : `PasswordAuthentication yes`
- Add autorized keys : `~/.ssh/authorized_keys`
- [Windows to Linux SSH](http://www.codeproject.com/Articles/497728/HowplustoplusUseplusSSHplustoplusAccessplusaplusLi)

Some useful applications
------------------------

-   [ZSH](https://wiki.archlinux.org/index.php/zsh) +
    [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
-   [Ranger](https://wiki.archlinux.org/index.php/ranger) (CLI file explorer)
    -   [Help with        commands](https://www.digitalocean.com/community/tutorials/installing-and-using-ranger-a-terminal-file-manager-on-a-ubuntu-vps)

- [Nice fonts](https://www.archlinux.org/packages/extra/any/adobe-source-sans-pro-fonts/) : `adobe-source-sans-pro-fonts`
- [Basics Fonts](https://www.archlinux.org/packages/extra/any/ttf-dejavu/) : `ttf-dejavu `

- Network manager : `networkmanager`
  - Applet : `network-manager-applet`

- Cairo-dock : `cairo-dock` + `cairo-dock-plugins`

- Locomotive for `ls` typo: `sl`, shows a locomotive when you type `sl` instead of `ls`

- `synergy` : Tool pour utiliser la souris & keyboard d'un seul PC (requiert `qt5-base`)

- `conky` + `conky-manager` : Show data on the desktop

- `libreoffice` : Office apps for Linux

- `openshot` : Video editing software
- `kdenlive` : Video editing software

- `parted` & `gparted` : Disk management tool

- `xarchiver` : Archiver tool (used by Thunar for example)

- `powertop` : Power management tool

- `lxmed` : LXDE Menu Editor (works for XFCE4 too)

- `okular` : PDF Viewer (based on KDE)

- `evince` : PDF Viewer (based on GNOME)

- `canto-daemon` : [Canto](http://codezen.org/canto-ng/) : CLI RSS feed aggregator. `canto-curses` : interface

- `insync` : Google Drive client (not free)

- `grive` : Google drive client (free) + `grive-tools` (**NOTE**: Make a SYMLINK to change the Google Drive directory, since it cannot be changed).

- `quicktile` : Python app/script to tile windows

- `clementine` : Music player

- `google-drive-ocamlfuse` : Google drive Client / Filesystem?

- `synapse` : Finder application (à la Launchy)
  - `bc` : Calculator plugin

- `xfce4-appmenu-plugin` : OSX style menu

- `geeknote-improved-git` : Evernote CLI client.
		- Note: `geeknote-git` is no longer updated, get the improved one.

### VirtualBox

- `virtualbox`

#### Error : Kernel driver not installed (rc=-1908) 

Solution found [here](http://unix.stackexchange.com/questions/106271/error-running-virtual-box-on-arch-linux)

I checked and I already had installed `virtualbox-host-modules` , then I tried to
reinstall it. That wasn't enough. Then I tried the command `sudo dkms autoinstall`. Dkms
isn't installed by default so I had to install `dkms package`. Then I started the service
by typing  `sudo systemctl enable dkms.service` . Then I could try with `sudo dkms
autoinstall` again. I tried starting virtualbox again but I still got an error. Then I
tried to manually load to module `vboxdrv` by typing `modprobe vboxdrv`. Now virtualbox is
working.

#### No Host-only adapter

If the only choice is `Not selected`, it means we need to add a new one.
To do so, go to `File > Preferences > Network > Host-only Networks` and add a new one. 

### WINE

- `wine`

#### No sound

- https://bbs.archlinux.org/viewtopic.php?id=135032
	Need some drivers for ALSA = `lib32-alsa-plugins lib32-libpulse lib32-openal`

#### PlayOnLinux

- `playonlinux`
- GUI to use WINE.

#### Remove everything from WINE

http://wiki.winehq.org/FAQ#uninstall


You can remove your virtual Windows installation and start from scratch by removing the hidden .wine directory in your user's home directory. This will remove all of your Wine settings and Windows applications.

To delete the .wine directory (known as a wineprefix), carefully paste the following commands into a terminal:


    cd
    rm -rf .wine
You may also rename it instead of deleting it, in case you want to keep a backup:


    mv ~/.wine ~/.wine-old

Your Windows applications, though deleted, will remain in your system menu. (Remaining desktop files and icons are located in `~/.local/share`):

To remove these leftover menu entries, carefully paste the following commands into a terminal:


    rm -f ~/.config/menus/applications-merged/wine*
    rm -rf ~/.local/share/applications/wine
    rm -f ~/.local/share/desktop-directories/wine*
    rm -f ~/.local/share/icons/????_*.{xpm,png}
    rm -f ~/.local/share/icons/*-x-wine-*.{xpm,png}
    rm -f ~/.local/share/icons/hicolor/*/*/application*.png
    rm -f ~/.local/share/icons/hicolor/*/*/????_*.{xpm,png}

#### Remove apps

http://wiki.winehq.org/FAQ#uninstall_app

To clean Open With List, please carefully paste the following commands into a terminal:

    rm -f ~/.local/share/mime/packages/x-wine*
    rm -f ~/.local/share/applications/wine-extension*
    rm -f ~/.local/share/icons/hicolor/*/*/application-x-wine-extension*
    rm -f ~/.local/share/mime/application/x-wine-extension*

[another link](http://ubuntuforums.org/showthread.php?t=1500338)

Note that Wine does not fully implement everything required to cleanly uninstall all applications. Some uninstallers might not function at all. To remove all programs installed under Wine, remove the wineprefix (usually the `~/.wine` directory):
Please note that in the following commands there should be no spaces in the path, particularly between `$HOME/` and `.whatever`.

    rm -rf $HOME/.wine

The uninstaller should remove menu and desktop entries. If the application was installed with an old version of Wine, it may not remove them. To remove all Wine-created menu entries run the following commands

`rm -f $HOME/.config/menus/applications-merged/wine*`

`rm -rf $HOME/.local/share/applications/wine`

`rm -f $HOME/.local/share/desktop-directories/wine*`

`rm -f $HOME/.local/share/icons/????_*.xpm`



### Themes Apps

- `gtk-engine-murrine` & `gtk-engine-unico` (engines)

- `faenza-icon-theme` : Nice icons (from [here](http://www.deviantart.com/art/Faenza-Icons-173323228))
- `faience-icon-theme` : Nice icons (from [here](http://desm0tes.deviantart.com/art/Faience-icon-theme-255099649))
-  `moka-icon-theme-git` : Icons (from [here](http://mokaproject.com/moka-icon-theme/)))
- `nitrux-icon-theme` : Icons (from [here](http://desm0tes.deviantart.com/art/Nitrux-293634207))
- `ultra-flat-icons` : Nice icon theme (on AUR & also [here](http://gnome-look.org/content/show.php/Ultra-Flat-Icons?content=167477))
- `numix-icon-theme-git` & `numix-circle-icon-theme-git` : Nice simple flat theme (from [here](https://aur.archlinux.org/packages/numix-circle-icon-theme-git/))
### Themes
#### To see the themes (XFCE) add the folder to `~/.themes`

- `Dorian theme` : Nice dark theme (from [here](http://killhellokitty.deviantart.com/art/dorian-theme-3-12-13-10022014-453111725))

#### Check out

-   Thunar (file manager)
-   [Volume icon for Laptop](https://www.archlinux.org/packages/community/i686/volumeicon/)

AUR
---

-   May need some more packages to get packages from the AUR
    -   Check https://www.youtube.com/watch?v=DAmXKDJ3D7M as around 4:45


- Dockbarx : `dockbarx`
    - XFCE dockbarx : `xfce4-dockbarx-plugin`
- Gnome Terminal Transparent : `gnome-terminal-transparent`

- Gnome Virtual File System : `gvfs`, for cd / external drive mounting

- Google calendar CLI : `gcalcli`

Dotfiles setup
--------------

-   [My dotfiles repo](https://github.com/jpgdev/dotfiles)
-   [Tutorial on dotfiles    management](http://blog.smalleycreative.com/tutorials/using-git-and-github-to-manage-your-dotfiles/)

Installing on a USB Key
-----------------------

- [Wiki Page](
  https://wiki.archlinux.org/index.php/Installing_Arch_Linux_on_a_USB_key
  )

Installing a Desktop Environment / Window Manager
-------------------------------------------------

**[Interesting link to install Desktop Environment](http://cyrilrose.com/easy-way-to-install-arch-linux-with-a-gui-frontend-xfce-or-enlightenment-on-an-old-laptop-uses-aui-script/)**

### Xorg

**This is required to use any graphical environment**

-   Installing [xorg-server](https://wiki.archlinux.org/index.php/Xorg)

    -   `pacman -S xorg-server xorg-xinit`
    -   `xorg-server-utils` can be installed too.
-   Optional dependancies
    -   `xterm` : terminal
    -   `xorg-twm` : basic window manager
-   Trackpad drivers (laptop)

    -   `pacman -S xf86-input-synaptics`
-   Installing GFX card drivers

    -   Identify GFX card
        -   `lspci | grep -e VGA -e 3D`
    -   Search for drivers
        -   `pacman -Ss xf86-video`
        -   or look in the `xorg-drivers` group (pacman)

    - For Optimus (Nvidia + Intel) use [Bumblebee](https://wiki.archlinux.org/index.php/Bumblebee).
      - Install the intel driver : [xf86-video-intel](https://wiki.archlinux.org/index.php/Intel_graphics)
      - May also need `lib32-mesa-libgl` from the multilib repository for 32-bit 3D support on x86_64,.
      - Install nvidia driver, either proprietary or open source (see [wiki](https://wiki.archlinux.org/index.php/Xorg#Driver_installation))

      - If there is no `.xinitrc` in the home directory, copy the one from `/etc/skel/.xinitrc` with the command: `cp /etc/skel/.xinitrc ~/.xinitrc`

    -   [VMWare](https://wiki.archlinux.org/index.php/Installing_Arch_Linux_in_VMware#In-kernel_drivers)
        -   `pacman -S xf86-input-vmmouse xf86-video-vmware svga-dri`
      -   **Troubleshooting:**
          -   [vmware(0) : failed to detect device screen object
            capability](http://superuser.com/questions/810355/cannot-run-x-in-archlinux-installed-in-vmplayer)
            (**NOT A FIX YET...**)

#### XFCE

-   Download and install XFCE

    -   `pacman -S xfce4`
    -   Install all members in the group
    -   libgl provider: chose **mesa-libgl**
-   Installing SLiM [Display
    Manager](https://wiki.archlinux.org/index.php/Display_manager)
    (Login window)
    -   `pacman -S slim`
    -   Enable SLiM
        -   `systemctl enable slim`

- `xfce4-goodies` : more plugins
- `xfce4-volumed` : used to fix the mute problem + adds a sound widget (like the luminosity one)
- `xfce4-screenshooter` : screenshoting utils.




###### Mute button bug

https://bugs.launchpad.net/xfce4-volumed/+bug/883485

1. create /etc/asound.conf with

        pcm.pulse {
          type pulse
        }
        ctl.pulse {
          type pulse
        }
        pcm.!default {
          type pulse
        }
        ctl.!default {
          type pulse
        }

2. And then to add a keyboard shortcut (via settings -> keyboard)
`XF86AudioMute` -> `amixer set Master toggle` ou `amixer -D pulse set Master Playback Switch toggle`

#### GNOME Shell

- `gnome-shell gnome-control-center`

TODO: Add missing ones ...

#### Enlightenment (e17)

-

#### Awesome window manager (awesome is the name)

- `awesome`
- `vicious`
- `alsa-utils`
- `wireless_tools`


#### OpenBox

http://wealsodocookies.com/posts/openbox-a-windows-environment-for-hackers/


- `openbox`
- `obconf` : configuration pour openbox
- `python2-xdg`
- `tint2` : simple task bar
- `volwheel` : simple volume option in tint2
- `xcompmgr` : composition (transparency)
- `docky` : Dock à la MacOS
- `gnome-terminal`

## Display Manager


### LightDM

- `light-locker` :
		To use : `light-locker-command --lock` (if using `xflock4` and it does not work, add the line in `/usr/bin/xflock4`).
		** NOTE: Need to do this each time the XFCE package for the lock (`xfce4-session` I think) is changed. **
- `xscreensaver` : screensaver for Light DM (and others) https://wiki.archlinux.org/index.php/Xscreensaver#Lightdm
- `lightdm-webkit-greeter` : Different greeter for LightDM (theme)

#### Changing the background


- Edit `/etc/lightdm/lightdm-gtk-greeter.conf` and change the line under `[greeter]` that starts with `background=` to the path to the file. It is recommended to put the file somewhere like `/usr/share/pixmaps` to be sure it is useable.
		- It is also required to make it readable like so: `sudo chmod +r name_of_the_file.jpg`

#### User switching not working with XFCE (maybe others)

[User switching under Xfce4](https://wiki.archlinux.org/index.php/LightDM#User_switching_under_Xfce4)

[Another link](https://forum.xfce.org/viewtopic.php?pid=32313#p34609)

- Create file at `/usr/bin/gdmflexiserver`

	```bash
	#!/bin/sh
	#pretend to be gdmflexiserver so the XFCE action plugin's Switch User function works

	#lock the current session
	xscreensaver-command -lock

	#then pass on the request to lightdm
	exec dm-tool switch-to-greeter
	```

- **Make sure the file is executable** : `sudo chmod +x /usr/local/bin/gdmflexiserver`
- The "xscreensaver" part is optional but is good for security.

## Troubleshooting


### Windows being a little bitch with shared partitions

- Run `ntfsfix /dev/sdX#` X# being the partition we want to fix.
- Then you can mount it manually and it should work
- **Do not forget to add `nofail` in the options in the fstab so arch will still boot event if this fails**



### Clock and date problems

The clock may use the UTC as default time instead of the localtime

https://wiki.archlinux.org/index.php/Time#Time_standard
