Below I collected some advices for installing Windows 10 on Lenovo Yoga 11E 6th gen.

The system as usually with Windows: installs without trouble (using USB created with Media Creation Tool), takes serial number from BIOS and, after connecting to WiFi, installs all drivers from Windows Update. Afterwards, some tweaks are needed:

# Graphics drivers
The Intel graphics drivers installed from Windows Update suffer from known bug, that causes memory leak in dwm.exe (desktop window manager) process. Once Windows update finishes its tasks, download the newer driver and install it manually. Make sure, the clean install option is marked. Note: installation takes a long time.
https://www.intel.com/content/www/us/en/support/articles/000058381/graphics.html

# Memory
This laptop has only 4GB of soldered RAM, therefore some usual tweaks neeed to be done once everything is installed:
- remove unnecessary autostart apps
- in the modern control panel, revoke background running permission for not needed apps (Yes... since Windows 8 I wonder, why the Calculator needs to run in the background :) ).
- if you know, what you're doing, you can review `services.msc`
- in the search box of start menu, search for performance, open "adjust the appearance and performance..." and choose best performance. You may keep some settings, like font smoothing.
- go to my folder `win_tweaks`, there are registry files with tweaks that I usually apply:
   - disable automatic install of advertised apps, like candy crush saga or whatever
   - disable searching the internet from start menu
   - if you intend to dual boot with Linux, you may want to set Windows to treat the hardware clock as UTC, which is default for Linux. Note: after applying that, reboot. After reboot, you will see incorrect time. Right click on system tray clock, adjust date-time and trigger internet synchronization.

# MS Edge
Using Microsoft Edge seems like a good choice on this laptop - it is fast and builtin. However, some tweaks can be done for performance:
- home page: in the top right corner there is a cogwheel icon. From there you can disable loading news, advertised links, dynamic wallpaper
- review the settings of Edge, there is a lot of them. You can disable edge autostart (can be also done from task manager Startup tab), disable running in background after Edge is closed. Also, you can disable MS rewards, sharing of browsing data with other applications and other nonsense options.

# WSL
If you want to use the WSL susbsystem, I recommend WSL1 instead of WSL2, it uses way less RAM. There may be a problem when using Visual Studio Code. For example, when installing Python addon to WSL, it terminates with EACCESS errors on folder renaming. The solution is: Open vscode config file, like `C:\Users\xxx\AppData\Roaming\Code\User\settings.json` and add the key:

```
{
   "remote.WSL.fileWatcher.polling": true
}
```

Close vscode, close all WSL processes `wsl --shutdown` and try again.

# Windows 11
The computer seems to support the minimum requirements for Windows 11 (4GB RAM, 64-bit dual core cpu, TPM 2.0, secure boot), but for keep the usable performance I have not tried it.
