Below I collected some advices for installing Kubuntu on Lenovo Yoga 11E 6th gen.
The advices are for Kubuntu in version 23.10 and 22.04 LTS, but one may find them useful for other Linux installation as well.

# Touchscreen cursor
To get the same mouse cursor behavior as in Windows (on screen touch, the cursor disappears, appears again after touchpad use):
- install packages `unclutter-xfixes` and `unclutter-startup`. unclutter-xfixes must be in version 1.6 or newer. If you are using Kubuntu 22.04 LTS, there is an older version of package. You can manually visit `packages.ubuntu.com` and download newer one from there (from newest release), then install with `sudo dpkg -i filename.deb`.
- open file `/etc/default/unclutter`. Locate line starting with EXTRA_OPTS and change the options in quotes to `"--hide-on-touch"`
- relogin

# Touchscreen in Firefox
Firefox does support the touchscreen events, but for some unknown reason they are disabled (unlike in Chrome). To enable them, one must set environment variable `MOZ_USE_XINPUT2=1`.
To set the variable globally in system, one can edit the file `/etc/security/pam_env.conf` and add the line:
`MOZ_USE_XINPUT2 DEFAULT=1`
then reboot.

# Touchscreen in Okular (PDF viewer) in LTS
The Okular version in Kubuntu LTS suffers from weird bug: touchscreen pinch zoom results in page scrolling. Easiest solution is to uninstall it and take newer one from Snap:
`sudo apt remove okular`, then 
`sudo snap install okular`
In 23.10 the program is already fixed.

# Speed up Firefox
There is a small but noticeable difference in performance of default Firefox (installed from Snap) compared to the one installed natively.
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

# Bad audio quality on internal speakers
This is one of the biggest unsolved issues with this laptop (Intel audio using Realtek ALC 257). On Windows, it features Dolby Audio Premium driver. On Linux, the sound is quiet and distorted. I do not have any ultimate solution for that.
The best what I can do:
- install easyeffects (in older LTS version: pulseeffects)
- install presets for easyeffects: https://github.com/JackHack96/EasyEffects-Presets
- try the Laptop aka LoudnessEqualizer preset.

What I also tried:
- I came across the method to record impulse response using Windows, creating IRS file out of it and then using it in EasyEffects convolver plugin. The method is described here: https://www.youtube.com/watch?v=Pktc2_Esp8I and here: https://github.com/shuhaowu/linux-thinkpad-speaker-improvements. With this method, sound is more "deep", however still quiet. My captured IRS is attached to this project, you can take it and try...
- JamesDSP is the simpler alternative to EasyEffects, seems to be more lightweight, does not create another audio sink (which changes volume controls in KDE). It is also capable of loading IRS.

Please contribute if you know any better solution or figured out a nice easyeffects preset.
