
Boot Chroot
===========

Boot debian/ubuntu based distributions installed in a chroot.

**The problem:** You like the flexibility of running various systems inside a chroot, but once in a while you need to boot from them. What do you do ?

**Option 1:** Move all the files into a partition and boot as usual (ughh).

**Option 2:** use this:

Run `make_chroot_initrd` script to create a new chroot-enabled initrd image from the existing one:

    #  ./make_chroot_initrd /chroot/trusty/boot/initrd.img-3.13.0-32-generic
    making new initrd: /chroot/trusty/boot/initrd.img-3.13.0-32-generic.chroot
    extracting
    patching
    packing
    done !

The new image will be exactly the same, except now it can handle a `chroot=` boot parameter.

With grub2 as bootloader you can add an entry to `/boot/grub/grub.cfg`:  
(or perhaps better `/etc/grub.d/40_custom`)

    menuentry "ubuntu trusty, (linux 3.13.0-32) (chroot)" {
    	insmod ext2	                  # or whatever you're using ...
    	set root='(hd0,7)'                # partition containing the chroot
    	set chroot='/chroot/trusty'       # chroot path
    	linux	$chroot/boot/vmlinuz-3.13.0-32-generic root=/dev/sda7 chroot=$chroot rw
    	initrd	$chroot/boot/initrd.img-3.13.0-32-generic.chroot
    }

(change files/partitions to match yours)

Done !

System-wide install
-------------------

Once you're happy with it you can make the changes permanent  
(until initramfs-tools package gets upgraded).  
In the chrooted system:

    # cd /usr/share/initramfs-tools
    # cp -pdrv .  ../initramfs-tools.orig	# backup
    # patch -p1 < path_to/boot_chroot/initrd.patch
    # rm *.orig */*.orig
    # update-initramfs -u

If `initrd.patch` does not work, especially for Ubuntu 16.04+,
please use `path_to/boot_chroot/initrd_1604.patch`.
From now on regular initrd image will support chroot booting.  
No need to use a separate initrd.chroot which may get out of sync with it then.


Notes
-----

Tested so far on Ubuntu (precise, trusty) and Debian (wheezy).

Once the initrd is done loading we're chrooted of course. Original partition is still available under `/host` if such directory exists.

Same approach should work for any linux distribution. The patch used by `make_chroot_initrd` will probably fail if the init scripts are too different, but shouldn't be too hard to adapt.

Why not use a loop mounted image instead of a chroot ?  
It's a lot more efficient cache & io-wise (unless using ploop)
