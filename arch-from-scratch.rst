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

.. code-block::

  # fdisk /dev/sda

  
  

  

Create the following file **/etc/wpa_supplicant/wpa_supplicant-wlan0.conf** assuming the previous command outputs **wlan0** as interface name with the following content:

.. code-block::

  ctrl_interface=/run/wpa_supplicant
  update_config=1

Now start wpa_supplicant with:

.. code-block::

  # wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
  
At this point run:

.. code-block::

  # wpa_cli -i wlan0

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
