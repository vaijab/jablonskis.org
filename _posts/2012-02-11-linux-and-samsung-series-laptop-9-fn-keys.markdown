---
date: 2012-02-11 22:20:16+00:00
layout: post
slug: linux-and-samsung-series-laptop-9-fn-keys
comments: true
title: Linux And Samsung Series 9 Laptop Fn Keys
categories:
- laptop
- linux
- tricks
- articles
tags:
- 90X3A
- function_keys
- howto
- keymap
- linux
- samsung
- tutorial
- udev_rules
- xorg
---

This is a follow up post to my [previous
post](http://jablonskis.org/2011/fedora-16-linux-on-samsung-series-9-np900x3a/)
on pretty much the same topic. The main reason I am writing again is that there
are still a lot of people confused how to get FN keys working properly and
there is a reason for people to be confused, because I did not mention about
how buggy Samsung's firmware is - once you press a function key, it does not
send a key release event back to the kernel, so we have to work around it.

I will try to get it right this time and will try my best to make the steps you
need to make as clear as possible.

Let's start with a little background information, so you know what's going on
under the hood. Everything starts with a key press, so on a key press your
keyboard sends a signal and linux kernel picks it up and this signal is known
as a scancode. The kernel has its own scancode to keycode mapping table,
so it maps a certain scancode to a keycode. You can look into
`/usr/include/linux/input.h` to see what your kernel uses for scancode to
keycode mapping - it is called a keymap.

I am sure those of you who tried to get Fn keys working have seen or configured
udev subsystem, so you might ask what role does udev play then? Well, udev does
not do much in this situation, but what it does is actually pretty crucial, so
we can plug almost any input device in and things will just work. So what udev
does in this situation is it re-maps scancodes to keycodes based on various
already pre-defined rules. The reason udev is important is there are so many
vendors out there and we all know how cool is to reinvent the wheel, so do
they. They just simply invent their own scancodes and this applies almost
entirely only to non-usb input devices (internal laptop keyboard is a non-usb
device), USB input devices have standard scancodes as far as I know (I might
be incorrect on this one).

Right, so the story goes something like this so far: [input device] -->
[kernel] --> [udev]. The next step is our X window system - Xorg. One may
think that Xorg just simply uses libudev to capture devices' events, but that
would be wrong. Xorg talks to the kernel and input devices directly and uses
its own key mapping tables, but by default it reads keymap table what is
current in the kernel on its startup, which then can be changed in the
userspace which will only be valid for that particular process of the Xorg
server.

So just to summarize that - kernel has preconfigured keymap table and it is
able to map most of the standard scancodes. The udev subsystem helps the
kernel to re-map any unusual vendor-specific scancodes to keycodes in the
kernel based on udev rules. On a startup Xorg reads the kernel keymap table.
Xorg never changes keymap table in the kernel directly, because Xorg has its
own keymap table which is called keysym table.

I hope the above gave you a little information so the below should be pretty
easy to understand. I assume you already have the working kernel module
compiled and loaded or the mainline kernel has the patches included for this
laptop if not check my previous
[post](http://jablonskis.org/2011/fedora-16-linux-on-samsung-series-9-np900x3a/).

I guess many of you skipped to this section right away :-). First you need to
make a keymap table for specific Fn keys. If you want to know how to write udev
rules and such, please google for it - there is definitely information on that.

* write keymap file for specific Fn keys

{% highlight bash %}
# /lib/udev/keymaps/samsung-90x3a

0x96 kbdillumup         # Fn+F8 keyboard backlit up
0x97 kbdillumdown       # Fn+F7 keyboard backlit down
0xD5 wlan               # Fn+F12 wifi on/off
0xCE prog1              # Fn+F1 performance mode (?)
0x8D prog2              # Fn+F6 battery life extender
{% endhighlight %}

If you wonder how to read scancodes, then I can tell you - it is pretty
simple. First make sure you are logged in as root to one of the consoles
(CTRL-ALT-F2 or similar). Then run the following and you should see similar
output (I assume your keyboard's input device is input/event4, if not sure,
then run `/lib/udev/findkeyboards`):

{% highlight bash %}
# /lib/udev/keymap -i input/event4
Press ESC to finish, or Control-C if this device is not your primary keyboard
scan code: 0xCE   key code: prog1
scan code: 0x89   key code: brightnessdown
scan code: 0x88   key code: brightnessup
scan code: 0x82   key code: switchvideomode
scan code: 0xF9   key code: f23
scan code: 0x8D   key code: prog2
scan code: 0x97   key code: kbdillumdown
scan code: 0x96   key code: kbdillumup
scan code: 0xA0   key code: mute
scan code: 0xAE   key code: volumedown
scan code: 0xB0   key code: volumeup
scan code: 0xD5   key code: wlan
scan code: 0x01   key code: esc
{% endhighlight %}
	
* write key press release file

{% highlight bash %}
# /lib/udev/keymaps/force-release/samsung-90x3a

# forces key release
0xCE # Fn+F8 keyboard backlit up
0x8D # Fn+F7 keyboard backlit down
0x97 # Fn+F12 wifi on/off
0x96 # Fn+F1 performance mode (?)
0xD5 # Fn+F6 battery life extender
{% endhighlight %}

The next step is to make use of those files by writing some udev rules.
* copy and paste the following rule below the other samsung related rules in that file
{% highlight bash %}
# /lib/udev/rules.d/95-keymap.rules

ENV{DMI_VENDOR}=="[sS][aA][mM][sS][uU][nN][gG]*", ATTR{[dmi/id]product_name}=="90X3A", RUN+="keymap $name samsung-90x3a"
{% endhighlight %}
	
* another rule this time for forced key release
{% highlight bash %}
# /lib/udev/rules.d/95-keyboard-force-release.rules

ENV{DMI_VENDOR}=="[sS][aA][mM][sS][uU][nN][gG]*", ATTR{[dmi/id]product_name}=="90X3A", RUN+="keyboard-force-release.sh $devpath samsung-90x3a"
{% endhighlight %}

* reload udev rules
`# udevadm control --reload-rules`

Now you should have udev rules setup and the Fn keys should just magically
work, well that be really cool, but unfortunately that's not the case. At least
some additional Fn keys work now, that is keyboard backlit control (Fn+F7 and
Fn+F8). But we still have wifi on/off, battery life extender and Fn+F1 (I
assume this is performance level switch from `normal` to `silent` and vice
versa). You are welcome to write your own wrapper script or use the one I
wrote. Place the script in your home directory like
`/home/username/bin/samctl.sh` and make it executable:

{% highlight bash %}
#!/bin/bash

#
# author: vaidas jablonskis <jablonskis at gmail dot com>
#
# script which allows to control wifi on/of, battery life extender,
# performance level for samsung series 9 laptop
#

# these paths should be correct by default
# if not set the variables correctly
batt_life_ext="/sys/devices/platform/samsung/battery_life_extender"
perf_level="/sys/devices/platform/samsung/performance_level"

# wlan rfkill name tends to change, so just to be safe
rfkill="$(grep -l "samsung-wlan" /sys/devices/platform/samsung/rfkill/rfkill*/name)"
if [[ -f "$rfkill" ]]; then
wlan_state="$(echo "$rfkill" | sed 's/name$/state/')"
fi

# function which toggles battery life extender on/off
batt() {
	batt_life_ext_value="$(cat $batt_life_ext)"
	if [[ $batt_life_ext_value -eq 0 ]]; then
	 echo "1" > $batt_life_ext
	else
	 echo "0" > $batt_life_ext
	fi
}

# function which toggles performance level (normal or silent)
perf() {
	perf_level_value="$(cat $perf_level)"
	if [[ "$perf_level_value" == "silent" ]]; then
	 echo "normal" > $perf_level
	elif [[ "$perf_level_value" == "normal" ]]; then
	 echo "silent" > $perf_level
	fi
}

# function which toggles wifi on/off
wlan() {
	wlan_state_value="$(cat $wlan_state)"
	if [[ $wlan_state_value -eq 0 ]]; then
	 echo "1" > $wlan_state
	else
	 echo "0" > $wlan_state
	fi
}

case "$1" in
	batt)
		batt
		;;
	perf)
		perf
		;;
	wlan)
		wlan
		;;
	*)
		echo "Usage: $0 {batt|perf|wlan}"
		exit 1
esac
{% endhighlight %}

Now the trickiest part, well at least it was for me. The default permissions
which are set to sysfs interface files which allow you to control various
things are too strict, that is that only root can write to them, one could
change their permissions to something 0666 or similar, so a regular user can
write to them, but we are not going to do that. It is insecure and lame :-)

I thought of something else - why can we not use sudo to run the script as
root? Sure, that should be straight forward. Add few lines to
`/etc/sudoers` file and off we go. Sounds easy, but there is a catch here. If
you bind a certain key to a "sudo script_name_cmd" for example this would not
work, because sudo requires a tty, well at least this is a default
requirement in Fedora and there is a very good reason behind this default. So
the script needs to be run by a terminal which allocates a pseaudo-tty but
then you run into another problem which is everytime you press an Fn key you
will see a terminal pop up for a split of a second, which might be annoying.

So let's setup our /etc/sudoers so that our specific commands can be run even
if there is no tty allocated, replace username with your user name:

{% highlight bash %}
Cmnd_Alias SAMCTL = /home/username/bin/samctl.sh batt, \
		    /home/username/bin/samctl.sh perf, \
		    /home/username/bin/samctl.sh wlan
Defaults!SAMCTL !requiretty
username	ALL=(ALL)	NOPASSWD: SAMCTL
{% endhighlight %}

Now there is the last bit left which is to bind Fn keys to the commands.
Whichever you prefer, either using gconftool-2 or using gnome3 GUI tool
"Keyboard --> Shortcuts --> Custom Shortcuts". Create three custom shortcuts,
in the command line put the following:

{% highlight bash %}
# wifi on/off
sudo /home/username/samctl.sh wlan

# battery life extender
sudo /home/username/samctl.sh batt

# performance level
/home/username/bin/samctl.sh perf
{% endhighlight %}

And that should be it. You may need to restart your Xorg session, but it might
not be necessary depending on how much copy+paste you did without actually
reading what I wrote above. But if you have gotten that far and still reading
this line, then good luck and post your issues in the comments section below, I
will try to help you out.

Also, I promise to file a bug report to udev people, so the rules and keymap
files will get included in the next release or so.
