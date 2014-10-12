---
date: 2011-12-23 02:54:18+00:00
layout: post
slug: fedora-16-linux-on-samsung-series-9-np900x3a
title: Fedora 16 Linux on Samsung Series 9 (NP900X3A) Laptop
categories:
- laptop
- linux
- tricks
- articles
tags:
- 90X3A
- backlit
- fedora
- fedora16
- keyboard
- laptop
- linux
- NP900X3A
- samsung
- series9
---

## Few words for a warm-up

Few months ago I bought myself a brand new and shiny
[ultrabook](http://www.samsung.com/uk/consumer/pc-peripherals/notebook-computers/ultra-portable/NP900X3A-A01UK?pid=uk_notebookcomputerstype_keyvisual1_np900x3a-a01uk_20110813)
from Samsung. It obviously came pre-installed with Windows7. I quickly rushed
to wipe the evil OS out, before I did an SSD clone, just in case I will ever
need Windows7 on this laptop again.

Right, so it was time to install my favourite linux distribution - fedora!
There was only fedora15 available back then, so I installed fedora15, which
I am not going to talk about much, because few weeks later fedora16 was
released, so I did a fresh install again.


## Installation

I built a bootable USB stick with fedora16 netinstall image on it, started the
installation and this is where the fun started.


### UEFI and Grub2

I knew my laptop had an option for `UEFI` firmware support, so I turned this
on, because UEFI is cool, right? Then I discovered that if one uses UEFI
subsystem, fedora falls back to use `grub-0.9x` rather than `grub2`,
there were some compatibility issues as far as I know, so I went for the
legacy BIOS option, because I really wanted to have grub2 booting my
OS.


### MSDOS Instead of GPT

Another issue I ran into was that Anaconda (fedora/rhel/centos etc) GUI
installer creates a GPT drive (I say drive because an SSD is not a disk)
label by default, which means that either BIOS or UEFI has to be able
to start a bootloader from a drive which has GPT partitition label. Obviously
proper UEFI implementation supports GPT with no problem, but apparently
both UEFI and BIOS implementation on this Samsung laptop are pretty bad
and do not support GPT drive labels (I tried them both to start a bootloader,
but unsuccessfully). Got it working? - please post a solution in the comments
below.

I chose the BIOS option and the old MSDOS drive (I know disk sounds better
here) label, which is kind of okay for me, I do not do anything too fancy with
my partition layout anyway.


### Partition alignment

This is pretty important step to do, usually anaconda does that for you, so
just ensure it has. Jump into a console Ctrl+Alt+F3 or something (I cannot
remember now off top of my head) and use parted align-check to check what
anaconda has done with your partitions :-)


### Successful Installation

The rest of the installation went really smoothly, there was a great
improvement of the anaconda installed compare to the older Fedora versions.
Right, install completed, rebooting the system and hope for the best - that
everything works out of the box.


## Post-install Fun

Like you probably experienced that yourself (otherwise it is very unlikely
you're reading this post) - not many things worked as expected.


### Working Pieces
	
* Multitouch Touchpad/Clickpad (had to change gnome3 settings to enable tapping etc)
* Screen brightness/backlight - Fn+F2 / Fn+F3
* Display switch - Fn+F4
* Touchpad on/off toggle - Fn+F5
* Sound VolUp/VolDown/Mute


### Not working Pieces

These are the most obvious things:
	
* keyboard backlit control - Fn+F7 / Fn+F8
* wifi on/off toggle button - Fn+F12
* Fn+F1 - not entirely sure what that does
* battery life extender - Fn+F6
* cpu fan was running pretty loud
* short battery life - approx 3 hours


### The Rest

The not working bits were pretty important to me and I wanted to get it fixed
asap, especially the keyboard backlit, which is a pretty cool feature to have.
The funny thing was, I could not control it. I figured that if one booted into
Windows7 prior fedora install and left the keyboard backlit on then it stayed
always on post the fedora install and vice versa. So in short - one had to boot
into windows adjust the keyboard backlit and then boot back to linux - that's
pretty cool :-)

Right, that's not what I wanted. So I dug a lot deeper.


## Solutions and Fixes

This is the most fun part for me at least. I was pretty glad that things turned
out to be this way. Unfortunately I am not a developer and cannot write a C
code, but I can barely read it.


### SSD and File System Tuning

Since this Samsung laptop has a [tiny (in physical size) 128GiB SSD
drive](http://thessdreview.com/our-reviews/samsung-pm800-128gb-msata-ssd-review-samsung-quietly-releases-another-top-tier-ssd/)
which is built from 20nm NAND flash, it deserves to be treated well by the
kernel and the file system, both to get a better performance and expand its
lifetime. Below there are few things I figured to be the best for my needs. I
will not explain why I chose those options, there is a lot of material online
which you can read about and see what suits you best.

* change the drive I/O scheduler to deadline

edit GRUB_CMDLINE in /etc/default/grub (I assume you use grub2) and add the
following option and then run `grub2-mkconfig > /boot/grub2/grub.cfg`:
`elevator=deadline`
	
* enable discard support on a filesystem and few other mount options

TRIM (discard) support is very important to maintain your SSD "healhty". These
are the mount option I chose to use on my ext4 filesystems:
`noatime,nodiratime,barrier=0,discard,data=writeback`


### Touchpad Delay

I forgot to mention, but I chose the Gnome3 aka gnome-shell desktop
environment, which as you already know happens to be not very user friendly
when it comes to customization and old good gnome2 menus and settings.

Anyway, my touchpad (or clickpad if you like) was working pretty well. The
laptop features a pretty massive touchpad, so it is very easy to touch it while
typing and it can get very frustrating, but there is an option gnome's "Mouse
and Touchpad" settings which allows you to disable the touchpad while typing,
but this gets tricky - it will not allow you to specify the timeout of the
delay, which is something ridiculous like 2 or 3 seconds.

Come on, seriously? Who wants to wait 2-3 seconds until you can use your
touchpad after you typed something? Last time I poked into the source code that
option was hardcoded - fair enough.

Not to worry, I have got a solution for that.

* Untick the box in the "Mouse and Touchpad" settings which says "Disable touchpad while typing"
* Write a simple script and place it in `~/bin/syndaemon.sh`

{% highlight bash %}
# enables custom synaptics touchpad settings
# start this with gnome-session

# enable touchpad after last keyboard key press delay
delay="0.5s"

syndaemon -d -R -k -i $delay
{% endhighlight %}

* Make it executable and add it to your session startup using `gnome-session-properties`

The script will start everytime your Gnome's sessions starts and it will launch
syndaemon (synaptics daemon). Feel free to poke around synclient too - it has
some cool tweaks you can apply to your touchpad.


### High CPU temp and Battery Life

This has something to do with a regression which was introduced with 3.0 or 3.1
linux kernel. I am not sure if I can call it a regression, because the option I
am going to list below were disabled due to some buggy hardware (not
necessarily Samsung laptops) AFAIK.

This laptop has Intel i5 dual core sandybridge CPU with integrated graphics
chip, which is known as i915 in the linux kernel, so adding few options to
the module during kernel boot time helps to solve high CPU temperature, noisy
fan and obviously battery life issues.

So do what you did with the drive I/O scheduler, just add the following options
to your kernel command line in `/etc/default/grub` and re-run
`grub2-mkconfig`:

`i915.i915_enable_rc6=1 i915.i915_enable_fbc=1 i915.lvds_downclock=1`

I noticed I get some sort of latency sometimes, especially noticeable on my
gnome-terminals, where it just won't take any input and feels like it freezes
for a second or so. I am not sure if this is related to the above options, but
the last time I looked at it I didn't see any syscalls or anything else which
could cause this weird freeze-latency issue, so I suspect it is that.

If you experience similar symptoms please let me know. (I have not tried to dig
deeper and investigate further).


### Keyboard Backlit/Backlight

Well, this is probably the most annoying issue I have experienced with my
laptop, but let me tell you that I have a solution for that. It is more like a
workaround for now, but I am sure this will become a proper solution.

First of all, the laptop is pretty new model and pretty pricey, so getting hold
of one is so easy I guess. We all know how much Samsung laptops suck on linux
support - they don't really care, they build hardware for the masses, not for
us poor linux people.

This took me a while to find a solution for. I knew there was a module called
samsung-laptop in the linux kernel which is maintained by **Greg K-H**. But
this module was not being loaded for some reason and the reason was that the
module checks the DMI product name etc against the hardcoded array of known
laptops in the module. So by looking in the source code of the module one can
see that there is not much code which could support keyboard backlit for this
laptop, so even if you force the module to load you wouldn't get success out of
it.

Fortunately there is an awesome developer called [Corentin
Chary](http://xf.iksaif.net/) who decided to contribute to samsung-laptop
module and wrote a number of patches for it. By looking at the
[conversation](http://thread.gmane.org/gmane.linux.drivers.platform.x86.devel/2700)
on kernel mailling-list these patches are being scheduled for 3.3 kernel
release, which is like months away - who wants to wait that long?

So I decided to grab the patches and compile the module against my current
running kernel. So this is briefly what I did (some common sense is always
welcome):

* clone Corentin's samsung-laptop git repo - [git://github.com/iksaif/samsung-laptop-dkms.git](git://github.com/iksaif/samsung-laptop-dkms.git)
* install current kernel headers: `kernel-devel` and `kernel-headers` packages
* obviously you will `gcc` and other development tools and libraries
* if the module compiles okay, load it: `insmod /path/to/compiled/module/samsung-laptop.ko`
* if it loads okay, copy it to `/lib/modules/$(uname -r)/kernel/drivers/platform/x86/samsung-laptop.ko`
* run `depmod -a`, now the module will load on the system boot

This is not it, what the module does is just adds support for various things
like keyboard backlight support, wifi toggle support (didn't have time to make
this work) and others in the kernel. But on a key press the kernel will scream
that it does not know what you mean by pressing the keys.

So you need to map the keycodes in the kernel so it knows about them. The
easiest way to do that is through the udev subsystem. Here is what you need to
do (I got it all figured for you):
	
* create a new udev keymaps file called `/lib/udev/keymaps/samsung-90x3a` with the below content:

{% highlight bash %}
0x96 kbdillumup # Fn+F8 - maps the scancode to a udev event
0x97 kbdillumdown # Fn+F7 - maps the scancode to a udev event
0xD5 wlan # Fn+F12 - this does not work
{% endhighlight %}
	
* edit the keymap rules file - `/lib/udev/rules.d/95-keymap.rules` and add the
  following line next before the other samsung laptop related lines (search for
  'samsung'):

{% highlight bash %}
ENV{DMI_VENDOR}=="[sS][aA][mM][sS][uU][nN][gG]*", ATTR{[dmi/id]product_name}=="90X3A", RUN+="keymap $name samsung-90x3a"
{% endhighlight %}

	
* load the newly created keymaps by running the following:
`/lib/udev/keymap input/event4 /lib/udev/keymaps/samsung-90x3a`

This loads the keymaps, so **udev** can instruct the kernel too. You need to
restart your X session so **gnome-settings-daemon** can digest the changes too.
But in theory you should be able to control your keyboard backlit using
`Fn+F7/Fn+F8`, if you cannot, reboot your OS just to double check that
everything works as expected.

I will send feature requests and fixes to udev developers as well as **Fedora**
people, so hopefully this gets added to the next release.


### Others

There are other function keys which don't work, but I suspect it is related to
gnome-settings-daemon not being able to act upon certain key strokes.


## Things to Poke

The new samsung-laptop module opens up a number of sysfs interfaces to
control your laptop. These interfaces are accessible via:


{% highlight bash %}
# ls -1 /sys/devices/platform/samsung/
battery_life_extender
leds
modalias
performance_level
power
rfkill
subsystem
uevent
usb_charge
{% endhighlight %}

They are self-explanatory, so go ahead and play with that. You can write some
scripts which control for example **battery_life_extender** or
**performance_level** which get executed on a certain key stroke. Custom
keyboard shortcuts can be set using Gnome keyboard GUI settings.

The is another interface to control screen brightness provided via **i915**
module which gives us a much wider range to control our screen brightness (poke
inside):

`/sys/class/backlight/intel_backlight/`

This also can be controlled using a simple script which can then be executed
using some keyboard shortcuts.

You could set `acpi_backlight=vendor` on kernel command line, so
gnome-settings-daemon will use the intel_backlight interface to control the
screen brightness, but don't - that will break your brightness control with
Corentin's patched module. It would work if you used samsung-laptop module
which is in the mainline linux kernel.


## Final Words

I guess that's it. Initially I thought I was going to write a short blog post
on how to fix the keyboard backlit control, but look what came out of my head,
way too much :-)

Hope this helps some people. Please post your experience with your ultrabooks
and give me a shout if you get into trouble solving your issues.

Thanks,
zooz
