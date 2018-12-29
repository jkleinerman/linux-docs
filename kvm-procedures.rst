
Enlarge an Image in KVM
-----------------------

Stop the virtual machine:

In the host do:

.. code-block::

  # qemu-img resize imagen.img +10G
  # modprobe nbd max_part=63
  # qemu-nbd -c /dev/nbd0 image.img
  # fdisk /dev/nbd0

|Delete de partition a regenerate with the new size.
|Once done the above, delete the ``nbd0`` device

.. code-block::

  # qemu-nbd -d /dev/nbd0

Start the machine and once inside, do:

.. code-block::

  # resize2fs /dev/vda1
