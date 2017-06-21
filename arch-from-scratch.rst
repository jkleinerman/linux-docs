Installing Arch From Scratch
=========================================================

.. contents::


Booting from a pen drive
-------------------------

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
  
  

Configure wpa_supplicant and systemd-networkd to connect to internet
--------------------------------------------------------------------

Install the following packages

.. code-block::

  # pacman -S wpa_supplicant iw



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
  Name=wlan0
  
  [Network]
  Address=10.10.7.71/24
  



Reboot the system to start the base system
------------------------------------------

.. code-block::

  # exit
  # umount -R /mnt
  # reboot







