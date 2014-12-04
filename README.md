
Boot Chroot
===========

Boot ubuntu-based distributions installed in a chroot.

**The problem:** You like the flexibility of running various ubuntu systems inside a chroot, but once in a while you need to boot from them. What do you do ?

**Option 1:** Move all the files into a partition and boot as usual (eeew).

**Option 2:** use this:

Run `make_chroot_initrd` script to create a new chroot-enabled initrd image from your existing one:

    #  ./make_chroot_initrd /chroot/trusty/boot/initrd.img-3.13.0-32-generic
    making new initrd: /chroot/trusty/boot/initrd.img-3.13.0-32-generic.chroot
    extracting
    patching
    packing
    done !

The new image will be exactly the same, except it can handle a `chroot=` boot parameter.

With grub2 as bootloader you can now add an entry to your `/boot/grub/grub.cfg`  
(or perhaps better `/etc/grub.d/40_custom`)

    menuentry "ubuntu trusty, (linux 3.13.0-32) (chroot)" {
    	insmod ext2
    	set root='(hd0,7)'                # change this to the partition containing the chroot
    	set chroot='/chroot/trusty'       # chroot path
    	linux	$chroot/boot/vmlinuz-3.13.0-32-generic root=/dev/sda7 chroot=$chroot rw
    	initrd	$chroot/boot/initrd.img-3.13.0-32-generic.chroot
    }

(make sure to change files / partitions to match yours)


Notes
-----

Same approach should work for any linux distribution. The patch used by `make_chroot_initrd` will probably fail if the init scripts are too different, but it shouldn't hard to adapt.
