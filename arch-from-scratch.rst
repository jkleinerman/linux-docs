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



