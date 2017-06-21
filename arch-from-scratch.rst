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


Mount the partition that would be used for root filesystem in /mnt:

.. code-block::

  # mount /dev/sda2 /mnt


Install the base packages:

.. code-block::

  # pacstrap /mnt base


Mount the EFI partition in /mnt/boot/efi:

.. code-block::

  # mkdir -p /mnt/boot/efi
  # mount /dev/sda1 /mnt/boot/efi


Generate the fstab file and check the resulting file:

.. code-block::

  # genfstab -U /mnt > /mnt/etc/fstab
 
  
Change root into the new system:

.. code-block::

  # arch-chroot /mnt
  

Set the time zone:

.. code-block::

  # ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
  

Run hwclock to generate /etc/adjtime:

.. code-block::

  # hwclock --systohc
  
  
Uncomment en_US.UTF-8 UTF-8 and es_AR.UTF-8 UTF-8 and other needed localizations in /etc/locale.gen, and generate them with:

.. code-block::

  # locale-gen
  
Set the LANG variable in ``/etc/locale.conf`` accordingly, for example:

.. code-block::

  LANG=en_US.UTF-8
  
Create the /etc/hostname file:

.. code-block::

  myhostname

Consider adding a matching entry to /etc/hosts:

.. code-block::

  127.0.0.1	localhost.localdomain	localhost
  ::1		localhost.localdomain	localhost
  127.0.1.1	myhostname.localdomain	myhostname
  
  
  
Set the root password:

.. code-block::

  # passwd
  
  
Install **GRUB**:

.. code-block::

  # pacman -S grub efibootmgr os-prober
  # grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub
  # grub-mkconfig -o /boot/grub/grub.cfg
  
  
Install wpa_supplicant package to be able to configure the wifi adapter when machine is rebooted:

.. code-block::

  # pacman -S wpa_supplicant

  
Reboot the system:

.. code-block::

  # exit
  # umount -R /mnt
  # reboot


Configure wpa_supplicant and systemd-networkd to connect to internet:



