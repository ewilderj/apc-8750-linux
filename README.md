apc-8750-linux
==============

Kernel configuration and build instructions for Linux on the Via APC 8750.

I assume you've installed Raspbian and wish to configure the kernel in order to add drivers, modules, etc.

Raspbian images and instructions can be found at http://www.raspbian.org/ApricotImages

My work is largely derived from that of Mike Thompson and Clément XXX from the APC forums. You can find a lot of information in the thread there: http://forum.apc.io/discussion/31/raspbian-port/

Linux kernel
============

Grab the kernel by cloning the APC 8750 repository from VIA:

    $ git clone git://github.com/apc-io/apc-8750.git

The repository as-is is set up for cross-compiling. If you wish to compile on the APC itself
(should be about 4 hours for a complete kernel and modules) then remove the CROSS_COMPILE setting
from the Makefile in the kernel root.

I'll assume you have some patience and are compiling this on the APC like I did.

Before compiling, I applied the patches from the Raspbian ApricotImages web page, you can
download those here: http://www.raspbian.org/ApricotImages?action=AttachFile&do=view&target=apc_apricot_r3_kernel.patch

Don't worry about patching the kernel config, throw away the .config file and use the one
in this repository.

To build the kernel

    $ make ubin

Then install the new kernel, making copies of the old modules in case you need to restore
them (I'm assuming you're running Raspbian off the SD card, so you can access the card to fix
things up.)

    $ sudo mv /lib/modules/2.6.32.9-default /lib/modules/2.6.32.9-default.orig
    $ sudo make modules_install
    $ sudo mv /boot/uzImage.bin /boot/uzImage.bin.old
    $ sudo cp uzImage.bin /boot

Notes about the kernel config
=============================

* IPv4 Netfilter is disabled. Via's engineers evidently had their source on a case-insensitive operating system at some point, so the ip_ECN and ip_ecn files in netfilter have clashed. I've not yet located the originals to rectify this error.
* By default CONFIG_ANDROID_PARANOID_NETWORK was set, I've turned this off. Otherwise most non-root network services won't run.


Getting Ralink RT5370 wifi dongle to work
=========================================

This is one of the recommended dongles from APC, but it won't just work with the drivers supplied in the kernel.
To make it work, you must do two things. Ensure the firmware-ralink package is installed, and then download and
compile the drivers from the compat-drivers project: https://backports.wiki.kernel.org/

I used compat-drivers-3.8-1, referenced from https://backports.wiki.kernel.org/index.php/Releases

Ensure you've made symlinks to the root of your kernel source tree to both /lib/modules/2.6.32.9-default/build and
/lib/modules/2.6.32.9-default/source.

Unpack the compat-drivers source and pick out the rt2x00 drivers

    $ ./scripts/driver-select rt2x00
    $ make && sudo make install

When installed, the modules will use the 'updates' directory and leave your old modules intact.
Reboot (or run 'make unload' to get rid of the old modules), and off you go.

Finally, ensure you've installed the right packages (assuming you're using WPA, because, who wouldn't?)

     $ sudo apt-get install wpasupplicant wireless-tools

And add a stanza to /etc/network/interfaces to configure the dongle:

    auto wlan0
    iface wlan0 inet dhcp
        wpa-ssid <YOUR-NETWORK-ESSID>
        wpa-psk <YOUR-NETWORK-PASSPHRASE>

