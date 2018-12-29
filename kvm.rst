
Enlarge an Image in KVM
-----------------------

Stop the virtual machine:

In the host do:

..code-block:



qemu-img resize imagen.img +10G
modprobe nbd max_part=63
qemu-nbd -c /dev/nbd0 image.img
fdisk /dev/nbd0

delete de partition a regenerate with the new size
qemu-nbd -d /dev/nbd0
