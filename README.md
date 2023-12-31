# Linux Surface Pro How-To

## Introduction
Microsoft tablets are nice machines, quite feature-rich, and of good quality! I recently needed an extra PC, and rather than just buying a new machine (it’s painful seeing the current prices for decent equipment these days), I remembered I had an old Surface Pro 4 that was lying unused. 

An old tablet, such as a Surface Pro 4, can be obtained for about £100 these days. I think these older tablets fly under the radar, but they are surprisingly versatile, usable in all manner of ways. A more recent tablet would be even more spectacular, but I used what I had. Mine came with a 2736 x 1824 touch-screen, an Intel i5 processor, 128 GByte SSD, and 4 GByte of RAM. All this is fine for running Windows, and normally I prefer a Windows desktop, but I had a use-case that required a Linux desktop and could also benefit from the tablet form-factor. If you have a similar need, this document shows how to go about it, step-by-step. By the end of it, for that £100 or so, you’ll have a fancy high-res Linux tablet with multi-finger capacitive touch, and Wacom-style pressure-sensitive pen capability.

<img width="100%" align="left" src="doc\mny.jpg">
.
 
## Buying a Surface Device
As mentioned, you should be able to pick one up for about £100. For instance, the advertisement below, at time of writing, shows one listed just a fraction more at £110 but with useful extras – an optional attachable keyboard, and pen.

<img width="100%" align="left" src="doc\tablet-ad.jpg">
.
 
If you’re obtaining a Surface device from eBay, then you’ll want to make sure it is supplied with a mains power brick, because the Microsoft Surface products use a magnetic connector; you probably will not want to use a non-genuine power brick, because (I’m speculating here; please rebut this if you have different thoughts!) there is a risk non-genuine ones apply power at all times to the connector, which will erode the contacts on the tablet during insertion/removal. The proper genuine Microsoft power brick sense when the connector is attached, and only then will it apply power, without risk of contacts arcing. Furthermore, it’s not generally a good idea to buy non-brand mains supplies, they can be dangerous.

## What Else is Required?
* Surface Tablet
* Mains Power Brick
* USB Hub
* SD Card
* Keyboard and Mouse

For initial setup, as well as the Surface tablet and its mains power brick, you’ll also need a USB hub (because some Surface devices only have a single USB port). I used a low-cost USB hub with integrated Ethernet socket. The Ethernet socket is handy as a backup but not essential since the Surface devices have WiFi capability. You will also need a USB memory stick; I used an 8 GB USB 2.0 memory stick (I don’t know if USB 3.0 will work, or if larger memory sizes are usable; I didn’t want to waste time troubleshooting, so I simply purchased a basic 8 GB USB 2.0 memory stick to dedicate to the task at hand.
Finally, a USB mouse and keyboard is needed for setup as well (they can be removed afterwards).

## Surface-izing Linux
Not all Linux distributions are Microsoft-Surface-friendly by default. For instance, the touchscreen didn’t work with the distribution I tried, and for me that was mandatory; it’s a key part of the value of the Surface tablet, to be able to use it without a keyboard attached occasionally.  Fortunately, there is a [linux-surface Github repo](https://github.com/linux-surface/linux-surface) with lots of valuable information. I used that information, with some slight modifications. The repo also offers pre-packaged releases, but I took the longer road and built the kernel (I needed this for future projects too, and I didn’t want to be stuck with a pre-built kernel); you can save yourself some effort and not do what I did if you don’t need to. All the steps I took are detailed below.

## Choose a Linux Distribution
Although more familiar with Ubuntu, I went with Debian simply because it seemed there was more information online for some of the steps I would need to perform to get it all running. I downloaded the latest Debian 12 ISO file. If you try the information here and have success with other distributions or releases, and have any tips, please let me know!

## Install the Linux Distribution
I plugged a USB stick into my desktop PC, and ran the excellent [Ventoy software](https://www.ventoy.net/en/download.html). Ventoy is used to format the USB stick into two partitions, where the smaller partition is bootable, and the larger partition can house ISO files. Once Ventoy had set up the USB memory stick (I didn’t change any defaults that Ventoy selected), I drag-and-dropped the Debian ISO file onto the USB partition.

The Surface Pro 4 only has one USB socket, so a USB hub was used to attach the USB memory stick plus a keyboard and mouse. There are buttons on the top edge of the tablet. Hold down the volume-up button and then press the power button on the tablet, and then release the volume-up button a few seconds later. Once the BIOS appears., disable secure boot, and then select the boot page, and then drag-left over the USB selection, to force boot from USB. The BIOS will restart, then you need to exit from it, for the USB boot to occur. Ventoy does its thing and allows you to select the ISO file to install Debian.

I had a 128 GByte SSD installed in the Surface Pro 4, and I partitioned the drive as follows during the Debian installation. You could of course accept the defaults, I just happen to think the allocations below are more suitable all-round.
```
#1 536. MB B f ESP (this was the default)
#2 30 GB f ext4 /
#3 8 GB f ext4 /var
#4 6 GB f swap swap
#5 3 GB f ext4 /tmp
#6 80.3 GB f ext4 /home
```

## Add Sudo Capability for the User
As root user, type the following (use your username instead of mine!):
`/usr/sbin/usermod -aG sudo shabaz`

Now exit out of root user, and exit out of the terminal (technically known as the shell), and then re-open a new terminal, for it to take effect. 
Unless any of the steps below explicitly mention being a root user, you don’t need to use su or sudo.

## Download the Linux Surface Patch
From the ~/development folder, download the Linux Surface patch repository:
`git clone https://github.com/linux-surface/linux-surface.git`

I went into the linux-surface folder, and made a note of which versions of Linux kernel are supported by the patches; at the time of writing, the versions happened to be 6.1 and 6.6.

## Download the Linux Kernel Source
I created a development folder from my home folder, i.e. a path like `/home/shabaz/development`. In Linux world, you can refer to your home folder as `~` so the path can also be written as `~/development`

In that folder, as root user, type:
```
apt-get install git
apt-get install rsync
apt-get install pahole
apt install build-essential binutils-dev libncurses5-dev libssl-dev ccache bison flex libelf-dev
```
Then, type the following in the `~/development` folder:
```
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
```

## Checkout a Linux Kernel release
I typed `uname -r` to see what kernel version I was running; it was `6.1.0-16-amd64`

I then typed `git tag` to see what releases I could choose. There’s a lot listed, so you could instead type `git tag | grep v6.1` to reduce the results. The latest one for 6.1 was `v6.1.69`, so I picked that:
`git checkout v6.1.69`

I created my own branch with any chosen custom name, by typing:
`git switch -c v6.1.69-surface`

## Apply the Linux Surface Patch
I went into the `~/development/linux` folder where the kernel source is, and then typed the following to apply the patch:
```
for i in ~/development/linux-surface/patches/6.1/*.patch; do patch -p1 < $i; done
```

Note that during the patch installation, you will likely get quite a lot of warnings about some files already existing. I accepted the default `[n]` option each time.

## Linux Kernel Configuration
To build the kernel, it first needs to be configured to suit your machine. Since you already have Debian installed, you can copy the configuration into the kernel source.

Look inside the `/boot` folder to see what configuration files are there; they will have names beginning with `config-`. From `uname -r` I already knew what kernel version was running, so I chose the config name that was closest. Type the following in the ~/development/linux folder:
```
./scripts/kconfig/merge_config.sh /boot/config-6.1.0.16-amd64 ~/linux-surface/configs/surface-6.1.config
```

## Build the Linux Kernel
Find out how many processor cores are available for the system to use:
`getconf _NPROCESSORS_ONLN`

My Surface Pro 4 had 4 cores available.

Type the following to build the Linux kernel:
```
make -j 4 deb-pkg LOCALVERSION=-linux-surface
```
If all went well, then (about 2.5 hours later!) the built `.deb` files will appear in the `~development` folder. You should see four `.deb` files created, although only three will be used (one is a debug image).

If you don’t see this, then there may have build errors to resolve. They will probably have scrolled off the screen. To see the errors, issue the make command as before. This time around less output will be generated (since the successfully built portions will not be rebuilt) and hopefully the error will be visible, all ready for googling to figure out what went wrong (This Is The Way!).

## Install the Linux Kernel
Type the following to become root user with particular `sbin` paths in your PATH environment variable:
`su -`

Now go to the development folder (replace my name below with your username!):
`cd ~shabaz/development`

Then, continuing as root user, type the following (copy-paste the filenames from the listed filenames):
```
dpkg -i linux-headers-6.1.69-linux-surface_6.1.69-linux-surface-4_amd64.deb linux-image-6.1.69-linux-surface_6.1.69-linux-surface-4_amd64.deb linux-libc-dev_6.1.69-linux-surface-4_amd64.deb
```
You may get a few warnings, I saw ones related to possible missing firmware.

Time to reboot. Type reboot!

If all goes well, touch should work! That’s the first sign that things are going well : )

Type `uname -r` in a terminal window, and you should see:
`6.1.69-linux-surface`

This is great that the touchscreen now works, but if you have a pen (the Surface tablets support a Surface Pen) that won’t work, so you need to further configure your Linux system to include specific drivers as discussed below.

## Install Linux Drivers and Configure Settings
In a terminal, type:
```
wget -qO - https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc | gpg --dearmor | sudo dd of=/etc/apt/trusted.gpg.d/linux-surface.gpg
```
Now type:
```
echo "deb [arch=amd64] https://pkg.surfacelinux.com/debian release main" | sudo tee /etc/apt/sources.list.d/linux-surface.list
```
As root user, type:
```
apt update
apt install libwacom-surface iptsd
```
Now reboot the machine! When it comes back, you should be able to add your Surface pen (if you have one) using Bluetooth, and it should also be detectable on the screen immediately too.

To test further, I installed a drawing package (as root user, type `apt install krita` and then it should appear as an installed app in the graphical environment). The pen worked great for sketches (the pressure-sensitiveness functioned) and multitouch with fingers worked in the app too, for pinch-and-zoom for instance.

One more thing that is useful is the on-screen keyboard, for the times that you want to use the tablet with no external keyboard. Go into the settings in Debian (it is in the graphical interface; a cog icon represents Settings) and then go to Accessibility, and turn on Always Show Accessibility Menu. By doing that, you’ll see a man icon appear top-right next to the network and sound icons. Whenever you need the on-screen keyboard, tap the icon and then tap Screen Keyboard. Now the screen keyboard will pop up automatically whenever text entry is available.

## Summary
For  about £100, it is possible to have a really nice tablet form-factor Linux machine. The steps are not difficult to perform; I took the slightly longer road by building the Linux Kernel rather than using a pre-built one, but it can sometimes be good to see what’s involved, even if you don’t currently have a use-case for building the kernel.

If you have any suggestions/ideas, or If you attempt it, it would be great to hear your feedback, and any tips/tricks you have discovered.

Thanks for reading!
