Installing Arch From Scratch
=========================================================

.. contents::


Booting from a pen drive
------------------------

Create the bootable pendrive with the following command

.. code-block::

  # dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress && sync

When booting, select the first option to boot in EFI mode.
Once booted, if the fonts looks very small, press "e", and add "nomodeset" at the end of kernel parametters.


Connect to intertnet using wifi adapter
---------------------------------------

Call wifi-menu wizard

.. code-block::

  # wifi-menu wlp2s0
  

Partition and format your hard disk drive
-----------------------------------------

Create the gpt using parted. (Windows need it)

.. code-block::

  (parted) mklabel
           gpt
 
Now partition the disk using fdisk. (It is easier)
 
.. code-block::

  # fdisk /dev/sda
  
- Create an EFI partition
- Create the root partition
- Create the swap partiotion
- Create the Windows partition

Create the filesystem on partitions for the EFI partition, root partition and swap.

.. code-block::

  # mkfs.fat -F32 /dev/sda1
  # mkfs.ext4 /dev/sda2
  # mkswap /dev/sda3
  
Activate the swap partition:

.. code-block::

  # swapon /dev/sda3


Install the base packages in your hard drive
--------------------------------------------

Mount the partition that would be used for root filesystem in ``/mnt``:

.. code-block::

  # mount /dev/sda2 /mnt

Select a good Server for the mirrorlist file ``/etc/pacman.d/mirrorlist``

Install the base packages:

.. code-block::

  # pacstrap /mnt base


Mount the EFI partition in ``/mnt/boot/efi``:

.. code-block::

  # mkdir -p /mnt/boot/efi
  # mount /dev/sda1 /mnt/boot/efi


Generate the fstab file and check the resulting file:

.. code-block::

  # genfstab -U /mnt > /mnt/etc/fstab
 

Configure time zone, clock, locale varaibles and hostname
---------------------------------------------------------


Change root into the new system:

.. code-block::

  # arch-chroot /mnt
  

Set the time zone:

.. code-block::

  # ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
  

Run hwclock to generate /etc/adjtime:

.. code-block::

  # hwclock --systohc
  
  
Uncomment ``en_US.UTF-8 UTF-8`` and ``es_AR.UTF-8 UTF-8`` and other needed localizations in ``/etc/locale.gen``, and generate them with:

.. code-block::

  # locale-gen
  
Set the LANG variable in ``/etc/locale.conf`` accordingly, for example:

.. code-block::

  LANG=en_US.UTF-8
  
Create the ``/etc/hostname`` file:

.. code-block::

  myhostname

Consider adding a matching entry to ``/etc/hosts``:

.. code-block::

  127.0.0.1	localhost.localdomain	localhost
  ::1		localhost.localdomain	localhost
  127.0.1.1	myhostname.localdomain	myhostname
  
  
  
Set the root password:

.. code-block::

  # passwd
  

Installation of **GRUB** bootloader:
------------------------------------

Assuming you have an EFI motherboard, install grub in the following way:

.. code-block::

  # pacman -S grub efibootmgr os-prober
  # grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub
  # grub-mkconfig -o /boot/grub/grub.cfg
  
  
Aditional packages before rebooting your new system
---------------------------------------------------

Install the following packages before rebooting and start your new base system:


To use wifi without the network manager:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Just install the package to have it when you reboot the system, but the configuration should be done after rebooting because there are problems when you try to run wpa_supplicant in a chrooted system:

.. code-block::

  # pacman -S wpa_supplicant iw
 
 
Vim editor
~~~~~~~~~~

.. code-block::

  # pacman -S vim
  
Make ``vi`` command call ``vim`` editor. This is neccesary for some commands like ``visudo``
 
.. code-block::

  # rm /usr/bin/vi
  # ln -s /usr/bin/vim /usr/bin/vi
  

Vim configuration file

.. code-block::
  
  # cp /usr/share/vim/vim80/vimrc_example.vim /etc/vimrc
	
To the previous file, add the following:

.. code-block::

  set tabstop=4
  set shiftwidth=4
  set expandtab
  set nobackup
  set noundofile
  set nowritebackup

  
To be able to paste text using the medium button of the mouse in a gnome-terminal, edit ``/usr/share/vim/vim80/defaults.vim`` and comment out the following lines:

.. code-block::

  "if has('mouse')
  "  set mouse=a
  "endif



Reboot the system to start the base system
------------------------------------------

.. code-block::

  # exit
  # umount -R /mnt
  # reboot


Set Enable NTP
--------------

.. code-block::

   # timedatectl set-ntp true


Fine tunning of bashrc
----------------------

- Install **Bash Completion** package

.. code-block::

  # pacman -S bash-completion
  

- Install **colordiff** package

.. code-block::

  # pacman -S colordiff
  
  
- To have the files colorized according to the extension generate ``/etc/DIR_COLORS``
  
.. code-block::

  # dircolors -p > /etc/DIR_COLORS


- Copy ``/etc/skel/.bash_profile`` and ``/etc/skel/.bashrc`` to ``/root`` directory

- Add the following lines to your new ``/root/.bashrc`` file:

.. code-block::
  
  PS1='\[\e[1;31m\][\u@\h \W]\$\[\e[0m\] '
  
  [ -r /etc/DIR_COLORS ] && eval `dircolors /etc/DIR_COLORS`
  
  alias ls='ls --color=auto'
  alias grep='grep --color=auto'
  alias diff='colordiff'
  
  shopt -s histappend  #Avoid overwritting history file
  
  HISTSIZE=5000        #History lenght of actual session
  HISTFILESIZE=5000    #File history lenght
  
  
  # Colored Man Pages
  man() {
   env \
   LESS_TERMCAP_mb=$(printf "\e[1;31m") \
   LESS_TERMCAP_md=$(printf "\e[1;31m") \
   LESS_TERMCAP_me=$(printf "\e[0m") \
   LESS_TERMCAP_se=$(printf "\e[0m") \
   LESS_TERMCAP_so=$(printf "\e[1;44;33m") \
   LESS_TERMCAP_ue=$(printf "\e[0m") \
   LESS_TERMCAP_us=$(printf "\e[1;32m") \
   man "$@"
  }

- Do the same for each user of your laptop

.. code-block::

  # cd /root
  # cp .bashrc .bash_profile /home/jkleinerman/
  # chown jkleinerman:jkleinerman /home/jkleinerman/.bashrc 
  # chown jkleinerman:jkleinerman /home/jkleinerman/.bash_profile
  
- Change the color of the normal users prompt

.. code-block::

  PS1='\[\e[1;32m\][\u@\h \W]\$\[\e[0m\] '
  


Configure wpa_supplicant, systemd-networkd and systemd-resolvd to connect to internet
-------------------------------------------------------------------------------------


Configure wpa_supplicant
~~~~~~~~~~~~~~~~~~~~~~~~

Check the name of the wifi adapter you are going to use with the following command:

.. code-block::

  # ip link ls

Create the following file ``/etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf`` assuming the previous command outputs **wlp2s0** as interface name with the following content:

.. code-block::

  ctrl_interface=/run/wpa_supplicant
  update_config=1

Now start wpa_supplicant with:

.. code-block::

  # wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf
  
At this point run:

.. code-block::

  # wpa_cli -i wlp2s0

This will present an interactive prompt (>), which has tab completion and descriptions of completed commands.


Use the **scan** and **scan_results** commands to see the available networks:

.. code-block::

  > scan
  OK
  <3>CTRL-EVENT-SCAN-RESULTS

  > scan_results
  bssid / frequency / signal level / flags / ssid
  00:00:00:00:00:00 2462 -49 [WPA2-PSK-CCMP][ESS] MYSSID
  11:11:11:11:11:11 2437 -64 [WPA2-PSK-CCMP][ESS] ANOTHERSSID
 
To associate with MYSSID, add the network, set the credentials and enable it:

.. code-block::

  > add_network
  0

  > set_network 0 ssid "MYSSID"
  OK

  > set_network 0 psk "passphrase"
  OK
  
  > enable_network 0
  OK
  <3>CTRL-EVENT-SCAN-STARTED 
  <3>CTRL-EVENT-SCAN-RESULTS 
  <3>WPS-AP-AVAILABLE 
  <3>Trying to associate with 18:a6:f7:60:e6:02 (SSID='MYSSID' freq=2412 MHz)
  <3>Associated with 18:a6:f7:60:e6:02
  <3>WPA: Key negotiation completed with 18:a6:f7:60:e6:02 [PTK=CCMP GTK=TKIP]
  <3>CTRL-EVENT-CONNECTED - Connection to 18:a6:f7:60:e6:02 completed [id=0 id_str=]

Finally save this network in the configuration file:

.. code-block::

  > save_config
  OK
  

To check link status, use following command.

.. code-block::

  # iw dev interface link



Do not enable wireless at boot. Start it manually when you need it since we are going to install the netwrok manager. Use this just when you need access from the console and you don't have the network manager started.
Start it using the following command:

.. code-block::

  # systemctl start wpa_supplicant@wlp2s0
  
wpa_supplicant@.service - accepts the interface name as an argument and starts the wpa_supplicant daemon for this interface. It reads a ``/etc/wpa_supplicant/wpa_supplicant-interfacename.conf`` configuration file. For this reason the file in ``/etc/wpa_supplicant`` was named ``wpa_supplicant-wlp2s0.conf``



Setting the IP address trough systemd-networkd
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create the following file ``/etc/systemd/network/wlp2s0.network`` assuming your interface is **wlp2s0**:

.. code-block::

  [Match]
  Name=wlp2s0
  
  [Network]
  DHCP=ipv4
  

**systemd-resolved** is required only if you are specifying DNS entries in .network files or if you want to obtain DNS addresses from networkd's DHCP client. Alternatively you may manually manage /etc/resolv.conf.
If you are going to use it, delete or rename the existing file `/etc/resolv.conf` and create the following symbolic link:

.. code-block::

  # ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
  

Do not enable systemd-networkd neither systemd-resolved at boot. Start it manually when you need them since we are going to install netwrok manager. Use them just when you need internet access from the console and you don't have the network manager started.
  
Each time you want to connect to internet without network manager, you should start the following units:

.. code-block::

  # systemctl start wpa_supplicant@wlp2s0
  # systemctl start systemd-networkd
  # systemctl start systemd-resolvd


SSH Server
----------

.. code-block::

  # pacman -S openssh
  
Edit ``/etc/ssh/sshd_config`` and uncomment ``UseDNS no``

Start the service manually when you need it or enable it at startup using

.. code-block::

  # systemctl start sshd



Mirror List
-----------

Generate a good ``/etc/pacman.d/mirrorlist`` using the online generator at: https://www.archlinux.org/mirrorlist/



Configuring console fonts
-------------------------

Configuring GRUB fonts
~~~~~~~~~~~~~~~~~~~~~~

Edit ``/etc/default/grub`` file and set the following line:

.. code-block::

  # GRUB_GFX_MODE=1024x768x32
  
Regenerate the grub configuration running:

.. code-block::

  # grub-mkconfig -o /boot/grub/grub.cfg



Configuring vconsole fonts:
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Install the package ``terminus-font``:

.. code-block::

  # pacman -S terminus-font
  
Set the desired font using ``setfont`` command, you can see the available fonts in ``/usr/share/kbd/consolefonts/``

.. code-block::

  # setfont ter-v32b
  
Make this permanent setting it in the file ``/etc/vconsole.conf``

.. code-block::

  FONT=ter-v32b
  FONT_MAP=8859-2


Create your user
----------------

.. code-block::

  # useradd -m -s /bin/bash -c "Jorge Kleinerman" jkleinerman
  # passwd jkleinerman
  

Sudo for your user
------------------

.. code-block::

  # pacman -S sudo
  # usermod -aG wheel jkleinerman
  
Uncomment the following line of ``sudoers`` files using ``visudo`` command

.. code-block::

  %wheel ALL=(ALL) NOPASSWD: ALL


Development tools
-----------------

- Install ``base-devel`` in order to use the **Arch User Repository**

- Install ``git`` in order to clone ``aurinup.sh`` script

- If you don't have python interpreter installed yet install ``python`` package

- Install ``python-virtaulevn`` package

- Install ``python-pip`` package

- Install ``ipython`` package

- Install ``docker`` package

.. code-block::

  # pacman -S docker
  # usermod -aG docker jkleinerman
  
- Install ``docker-compose``

- Install ``retext``

.. code-block::

  # pacman -S retext python-docutils

Install Graphical Environment (Gnome)
-------------------------------------

1) Install ``gnome`` package

.. code-block::

  # pacman -S gnome

And select default options (hit Enter key 3 times)

2) Enable GDM:

.. code-block::

  # systemctl enable gdm.service
  # systemctl start gdm.service
  
3) Enable NetworkManager:

.. code-block::

  # systemctl enable NetworkManager.service
  # systemctl start NetworkManager.service
  

4) Set Bluetooth:

Start the ``bluetooth.service`` systemd unit. You can enable it to start automatically at boot time doing ``systemctl enable bluetooth.service``

5) Using the default gnome browser, go to extensions.gnome.org and download the following extensions

|   - **taskbar** by **zpydr**
|   - **transparent top bar**
|   - **dash to dock**

6) Install ``gnome-tweak-tool`` and manage the above extensions.

7) Install better fonts

.. code-block::

  $ sudo pacman -S ttf-dejavu
  
8) Install ``xfce4-terminal``

9) Install ``Firefox``


Google Chrome Browser
---------------------

Install google-chrome package from the AUR.

.. code-block::

   # ./aurinup.sh google-chrome


Install addblocks google chrome extension


LibreOffice
-----------

Install Libre Office package and the spelling corrector

.. code-block::
  
  # pacman -S libreoffice-still
  # pacman -S hunspell-es
  # pacman -S hunspell-en_US 


Tuning
------

Set the lock screen delay:

.. code-block::

  # By default it is 1 minute. Set delay time to 5 minutes
  $ gsettings get org.gnome.desktop.session idle-delay
  uint32 60
  $ gsettings set org.gnome.desktop.session idle-delay 300

Dropbox:

Install from AUR ``dropbox`` and ``nautilus-dropbox``. The last one is for Nautilus integration

Keyboard Accents:

Go to **Settings**, **Region & Language** and add **English (intl., with AltGr dead keys)** to **Input Sources**.  

Take into account that there is another layout which name is **English (US intl., with dead keys)**. Only the first one should be selected.

Enable Desktop Icons and Right click:

.. code-block::

  $ gsettings set org.gnome.desktop.background show-desktop-icons true
  
  
Enable H.264 for Gnome Videos:

.. code-block::

   # pacman -S gst-libav
   
Cups
----
.. code-block::

  # pacman -S cups
  # pacman enalbe cups-browsed.service
  # pacman start cups-browsed.service
  
Install the necessary driver if need it and configure cups in the url: localhost:631


Wrting on NTFS filesystems
--------------------------

.. code-block::

  # pacman -S ntfs-3g
  
  
Multimedia Software
-------------------

.. code-block::

  # pacman -S inkscape
  
Nautilus
--------

To sort directories before files do:

.. code-block::

  # pacman -S dconf-editor
  
Execute it and go to: ``org/gtk/settings/file-chooser/`` and enable ``sort-directories-first``


MySQLdb
-------

.. code-block::

  # pacman -S mariadb-clients
  # pip install mysqlclient

