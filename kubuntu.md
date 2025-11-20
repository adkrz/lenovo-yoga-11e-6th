Below I collected some advices for installing Kubuntu on Lenovo Yoga 11E 6th gen.
The advices are for Kubuntu in version 23.10 and 22.04 LTS, but one may find them useful for other Linux installation as well, like Linux Mint.

# During installation
Make sure, that you create a swap partition, at least 4GB. This will help with small amount of RAM + enable hibernation.
If forgotten, at least create a swap file:
```
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
mkswap /swapfile
```
Open `\etc\fstab` and add: `/swapfile swap swap defaults 0 0`, then reboot.
Note: swapfile option does not work with hibernation.

# Touchscreen cursor
To get the same mouse cursor behavior as in Windows (on screen touch, the cursor disappears, appears again after touchpad use):
- install packages `unclutter-xfixes` and `unclutter-startup`. unclutter-xfixes must be in version 1.6 or newer. If you are using Kubuntu 22.04 LTS, there is an older version of package. You can manually visit `packages.ubuntu.com` and download newer one from there (from newest release), then install with `sudo dpkg -i filename.deb`.
- open file `/etc/default/unclutter`. Locate line starting with EXTRA_OPTS and change the options in quotes to `"--hide-on-touch"`
- relogin
- or:
In Cinnamon, it works OOTB, as well as in KDE in Wayland mode

# Touchscreen in Firefox
Firefox does support the touchscreen events, but for some unknown reason they are disabled in X11 (unlike in Chrome). To enable them, one must set environment variable `MOZ_USE_XINPUT2=1`.
To set the variable globally in system, one can edit the file `/etc/security/pam_env.conf` and add the line:
`MOZ_USE_XINPUT2 DEFAULT=1`
then reboot.
In Wayland, touch gestures work by default.

# Touchscreen in Okular (PDF viewer) in LTS
The Okular version in Kubuntu LTS suffers from weird bug: touchscreen pinch zoom results in page scrolling. Easiest solution is to uninstall it and take newer one from Snap:
`sudo apt remove okular`, then 
`sudo snap install okular`
In 23.10 the program is already fixed.

# Speed up Firefox
There is a small but noticeable difference in performance of default Firefox (installed from Ubuntu's Snap) compared to the one installed natively.
You can follow the official guide from Mozilla:
https://support.mozilla.org/en-US/kb/install-firefox-linux#w_install-firefox-deb-package-for-debian-based-distributions
After that, migrate the profile (copy data from ~/snap/ to ~/.mozilla) and remove the old FF using `snap remove firefox`.

Note, that after upgrading 23.10 to 24.04 LTS version (because of end of support), the just added repository will be disabled by the updater. To re-enable it after installing new system version:
- do not do apt-get upgrade now, because this will restore the snap firefox
- go to /etc/apt/sources.list.d and remove the ".distUpgrade" suffix from "firefox..." file.
- apt-get update && apt-get upgrade

# No screensaver suppression in LXDE
If you decide to use LXDE instead of KDE, there is a problem: screen goes to sleep after inactivity, also when watching movie in browser or during Skype video call. This is a bug, it seems to affect all Chromium-based browsers and Electron apps (like Skype). I don't know any permanent solution. The working ones are:
- use KDE :)
- use Firefox, this browser suspends screensaver properly
- install Caffeine program and activate it before Skype session

# Microphone - records with loud noise
Either:
- open terminal, run alsamixer, switch to recording devices and set microphone boost to zero
- or, click the KDE system tray sound icon and lower microphone volume to around 32-35%

# Enable Bluetooth
By default, BT cannot be enabled from KDE control panel. The rfkill command reports it as hard blocked. Create file `/etc/modprobe.d/thinkpad_acpi.conf` with the following contents:
`options thinkpad_acpi dbg_wlswemul=1 wlsw_state=1`
, then reboot

# Screen rotation in convertible mode
https://askubuntu.com/questions/1469242/how-to-autorotate-convertible-laptop-screen-in-ubuntu-and-kubuntu
- in Wayland works out of the box
- in X11 there is an app called Screen Rotator
- In Cinnamon works out of the box

# In dual boot, time is off by several hours and fixes itself after about 1min after startup
- Set Windows installation to treat BIOS clock (RTC) as UTC. Import the following registry key:
```
﻿Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation]
     "RealTimeIsUniversal"=hex(b):01,00,00,00,00,00,00,00
```

- reboot the Windows. After reboot, clock will be invalid. Correct it e.g. by updating it from internet time
- reboot to Linux and type `sudo timedatectl set-local-rtc 0`

# Touching USB mouse wakes up from standby
If this is unintended behavior, follow the advice from [Mint Forum](https://forums.linuxmint.com/viewtopic.php?t=413168), to create service that commands USB to ignore wakeup:
```
lspci | grep USB | cut -d " " -f 1 > ~/t7s1 \
&& cat /proc/acpi/wakeup > ~/t7s2 && grep -f ~/t7s1 ~/t7s2 > ~/t7s3 \
&& cat ~/t7s3 | cut -c 1-4 > ~/t7s1 \
&& sed -i -e 's|^|echo "|' -e 's|$|" > /proc/acpi/wakeup|' ~/t7s1 \
&& tr -d '\t' < ~/t7s1 > ~/t7s2 && sed -i '1s|^|#!/bin/sh\n|' ~/t7s2 \
&& sudo cp -f ~/t7s2 /usr/local/bin/nowusb.sh \
&& sudo chmod +x /usr/local/bin/nowusb.sh && rm -f ~/t7s1 ~/t7s2 ~/t7s3 \
&& printf '%s\n' '[Unit]' 'Description=no-wakeup-usb' '[Service]' \
'ExecStart=/usr/local/bin/nowusb.sh' '[Install]' \
'WantedBy=multi-user.target' \
| sudo tee /etc/systemd/system/no_wakeup_usb.service \
&& sudo systemctl enable no_wakeup_usb.service
```

# Random screen flickering
The screen blinking issue on Intel GPUs is known. 
What seems to help is adding kernel parameter `intel_iommu=igfx_off` to `/etc/default/grub`, then calling `update-grub` and reboot.
The common solutions like adding `i915.enable_psr=0`, `xe.enable_psr=0` to kernel command line do not work.

# Hardware accelerated video playback
Seems to be supported in both Firefox and Chrome, in X11 and Wayland.
Please do not make any strange changes to `/etc/X11/xorg.conf.d` - these tend to mess up with VAAPI. If you must do it, make sure, that you use `modesetting` driver, not `intel` which is deprecated.
You may want to install `intel-media-va-driver-non-free` in order to support both encoding and decoding of video streams. The free version only supports decoding.
To verify if HW acceleration is enabled, check page `about:support` in Firefox and `chrome://gpu` in Chrome. Also, during playback, the command `sudo intel_gpu_top` from `intel-gpu-tools` package informs if the video acceleration is actually in use. This not only depends on driver support, but also on what codec does e.g. Youtube use currently.
The command `vainfo` informs about video decoding capabilities as well.

# Wayland in Mint
It is still experimental - expect troubles :)
Screensaver and screen locking is not working - disable it. Otherwise you may end up with screen randomly going blank. To "unlock" it, switch to console mode `CTRL+ALT+F1` and then back to GUI: `CTRL+ALT+F7`.
To run apps like Chrome or KeepassXC, you'll need to install `qtwayland5` package.

# Bad audio quality on internal speakers
This is one of the biggest unsolved issues with this laptop (Intel audio using Realtek ALC 257). On Windows, it features Dolby Audio Premium driver. On Linux, the sound is quiet and distorted. I do not have any ultimate solution for that.
The best what I can do:
- install easyeffects (in older LTS version: pulseeffects)
- install presets for easyeffects: https://github.com/JackHack96/EasyEffects-Presets
- try the Laptop aka LoudnessEqualizer preset.

What I also tried:
- I came across the method to record impulse response using Windows, creating IRS file out of it and then using it in EasyEffects convolver plugin. The method is described here: https://www.youtube.com/watch?v=Pktc2_Esp8I and here: https://github.com/shuhaowu/linux-thinkpad-speaker-improvements. With this method, sound is more "deep", however still quiet. My captured IRS is attached to this project, you can take it and try...
- the easyeffects filter chain: compressor + convolver + limiter seems to work the best
- JamesDSP is the simpler alternative to EasyEffects, seems to be more lightweight, does not create another audio sink (which changes volume controls in KDE). It is also capable of loading IRS.
- another alternative (more CPU-friendly) is to use Pipewire convolver directly - see file conv.conf in this repository. This file creates new virtual device "YogaHD" to use instead of default one.

Please contribute if you know any better solution or figured out a nice easyeffects preset.

