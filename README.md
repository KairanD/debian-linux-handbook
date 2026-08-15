# Debian Linux Handbook

- Written by: KairanD.
- Version: 1.1.2.
- Date: 2026/08/15.
- GNOME version: 48.
- License: CC BY-SA 4.0. You may share and adapt the content with attribution, and derivative works must be released under the same license.

I wrote this guide for people to easily replicate my Debian Linux GNOME install. It's also, of course, a place for me to keep instructions for future installs.

### Additional credits

The contents on the "resources" folder were downloaded from:
- 60-openrgb.rules: OpenRGB by Adam Honse (@CalcProgrammer1), available at: https://gitlab.com/CalcProgrammer1/OpenRGB

## Installation

### Download ISO

The Debian ISO can be downloaded on https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/. You can use applications such as Rufus, Balena Etcher and Impression to create the bootable USB.

### Configuring the installer

After booting, choose "graphical expert install". Follow the instructions below (if something is not mentioned, then just advance with the suggested default settings):

1. **Choose language:** select your preffered language to be used by the installer and the system. When asked, select the variation used in your country.
2. **Configure the keyboard:** select the layout used in your country. 
3. **Detect and mount installer media:** just hit enter and wait.
4. **Load components from installation media:** do not select anything. Advance.
5. **Detect network hardware:** just hit enter and wait.
6. **Configure the network:** choose your main network interface. If it's wireless, then enter the password. Insert a name for the computer when asked.
7. **Configure users and passwords:** do not allow login as root. Then create an user and set it with root permissions when asked.
8. **Configure clock:** select the region in your country to configure the clock accordingly.
9. **Detect disks:** just hit enter and wait.
10. **Partition disks:** select the manual method. Select the disk to create a new GPT partition table. First, create a 1 GiB partition, with the EFI name, and select the option to use it as EFI system partition. Also mark it as bootable. Then create a new partition using the remaining space, using a easy to understand name such as "SSD1". Continue. The installer will complain about the lack of a swap partition. Refuse when asked to go back to the partitioning menu. Then choose to write changes to disk.
11. **Install base system:** just hit enter using the defaults and wait.
12. **Configure the package manager:** select to use non-free firmwares and non-free applications. Don't activate APT source repositories.
13. **Select and install software:** use the spacebar to keep only default system utilitaries active.
14. **Install bootloader:** if your computer has an AMD64 UEFI (almost always the case for computers released after 2012), choose systemd-boot. For computers with mixed-mode UEFI (x86 UEFI with x64 processor), such as the Asus T100TA, choose GRUB.
15. **Finish the installation:** keep the defaults and continue.

### Installing the desktop environment and basic packages

After rebooting, the system will enter CLI mode. Enter your username and password and install the packages below. First, we need to ensure the system is updated:
```
sudo apt update && sudo apt upgrade
```
Then install a CLI system information tool, GNOME, a tweaks application, an extension to show tray icons, and Flatpak support (including a software store plugin):
```
sudo apt install fastfetch gnome-core gnome-tweaks gnome-shell-extension-appindicator gnome-software-plugin-flatpak
```
To have a dock, install the Dash to Dock extension:
```
sudo apt install gnome-shell-extension-dashtodock --no-install-recommends
```
Let's then add the UFW firewall with a graphical interface:
```
sudo apt install gufw
```
Finally, the Microsoft corefonts, for compatibility with documents created by Microsoft applications:
```
sudo apt install ttf-mscorefonts-installer
```
If you have an Epson printer, then install:
```
sudo apt install printer-driver-escpr
```
After installing everything, disable the network hotplug:
```
sudo nano /etc/network/interfaces
```
Just comment the last two lines with the name of the network and the password. It is necessary to use the graphical network menu.

### Activate Flathub

To add the Flathub repository, run this command on the Terminal application:
```
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

### Configure swapfile

I recommend using 1/4 of your available RAM, or at least 4 GiB. First, let's create the file ("8388608", in this example, means 8 GiB):
```
sudo dd if=/dev/zero of=/swapfile bs=1024 count=8388608 status=progress
```
A swapfile of 4 GiB would have half, it is, 4194304. Change the size accordingly:
```
sudo dd if=/dev/zero of=/swapfile bs=1024 count=4194304 status=progress
```
Then we need to set permissions, enable the swapfile and add it to the fstab file. Run each one of these commands, in sequence:
```
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Enable VRR support on GNOME

Debian 13 still uses GNOME 48. Variable refresh rate (VRR) is considered experimental, but works well. To enable the system option, run:
```
gsettings set org.gnome.mutter experimental-features "['variable-refresh-rate']"
```

### Extensions

First, download Extension Manager (Matthew Jakeman) from the Software application. Activate "AppIndicator and KStatusNotifierItem Support" and "Dash to Dock" if not already active.

If you're using a laptop, then you probably already have the brightness control active at the top right corner of your screen. However, for desktop users, it is currently necessary to use an extension. Browse and install "Brightness control using ddcutil" (themightydeity). Now open the Console application and do:
```
sudo apt install ddcutil
sudo modprobe i2c-dev
sudo cp /usr/share/ddcutil/data/60-ddcutil-i2c.rules /etc/udev/rules.d
sudo usermod $USER -aG i2c
echo 'i2c-dev' | sudo tee -a /etc/modules-load.d/i2c.conf
```
Now you'll have a working brightness control widget at your panel. Open its settings to configure the "button location" as "system menu".

### Hardware specific adjustments

#### Computers with modern Nvidia GPUs

If you use a Maxwell or Pascal Nvidia Graphics card (GTX 750, GTX 750 Ti, GTX 900 series or GTX 1000 series), install the Nvidia 550 proprietary driver:
```
sudo apt install nvidia-kernel-dkms nvidia-driver
```
If you use a recent Nvidia Graphics card, Turing (GTX 16xx or RTX 20xx) or above, then install the open 550 driver version:
```
apt install nvidia-open-kernel-dkms nvidia-driver
```
Note: Maxwell and Pascal support was ended by Nvidia. Your card will not work with proprietary drivers on newer Debian releases.

#### Computers with modern AMD GPUs

If you have a current AMD GPU, such as the RX 7600 XT, the best drivers (open source) are already installed. However, there may be spikes during idle that heat the card a little. If that's the case, do and reboot:
```
echo 'options amdgpu ppfeaturemask=0xFFFF7777' | sudo tee -a /etc/modprobe.d/99-amdgpu-overdrive.conf
sudo update-initramfs -u -k all
```

#### Laptops with old (and probably not very useful) Nvidia dedicated GPUs

Old Nvidia graphics cards can be a pain on Linux. If you have a laptop with one, such as the GT 740M my ASUS S46CB has, the best approach is to completely disable the card and use only integrated graphics. These cards are slow and their driver support has been terminated for a long time. Check my other repository and follow the instructions to disable yours: https://github.com/kairand/disable-nvidia-linux

## Applications

### Essential Flatpaks

These are the essential applications I always install on my computers.

* Extension Manager (Matthew Jakeman): downloads and updates GNOME extensions.
* Flatseal (Martin Abente Lahaye): manages Flatpak application's permitions.
* GIMP (The GIMP team): image manipulation application.
* LibreOffice (The Document Foundation): complete office suite.
* Mission Center (Mission Center Developers): shows usage of system resources and open processes.
* Rhythmbox (The Rhythmbox developers): complete music player for local files.
* VLC (VideoLAN et al.): the universal media player, with amazing compatibility.

### Other Flatpaks

These are other applications I use.

* Arduino IDE v2 (Arduino SA): default Arduino IDE.
* Bottles (The Bottles Contributors): wine prefixes manager.
* Boxes (The GNOME Project): virtual machines manager.
* Chromium (The Chromium Authors): alternative web browser.
* Descodificator (Bilal Elmoussaoui): reads and generates QR codes.
* Discord (Discord Inc.): Discord client.
* Earbud manager for Galaxy Buds (Tim Schneeberger): audio manager for Samsung's Galaxy Buds.
* Fragments (Felix Häcker): torrent downloader and manager.
* Galaxy Buds Manager (Tim Schneeberger): software for configuring Galaxy Buds specific features.
* GitHub Desktop (shiftkey): unofficial GitHub Linux client.
* Godots (Maxim Kovkel): manager for multiple versions of the Godot game engine editor.
* HydraPaper (Gabriele Musco): selection of different wallpapers for multiple monitors.
* Impression (Khaleel Al-Adhami): creates Linux boot drives.
* Krita (Krita Foundation): digital painting software with some image manipulations tools.
* LosslessCut (Mikael Finstad): quick and easy editor for video files.
* Minecraft (Mojang AB): official Minecraft launcher.
* Minion (Good Game Mods, LLC): addon manager for The Elder Scrolls Online.
* OpenRGB (Adam Honse, OpenRGB Team): manages RGB devices.
* PDF Arranger (The PDF Arranger team): edits and resizes PDF files.
* Protontricks (Janne Pulkkinen): software to manage Steam's Proton game prefixes.
* Solanum (Christopher Davis): pomodoro tracker to help with your tasks.
* SoundConverter (Gautier Portet): multi format audio files converter.
* Steam (Valve Corporation): Steam client.
* Steam Link (Valve Corporation): application to receive straming signal from a Steam client.
* Unity Hub (Unity Technologies): downloads and install versions of the Unity Editor for game making.
* Video Downloader (Unrud): downloads video and audio from YouTube and other sources.
* VSCodium (The VSCodium team): VSCode without Microsoft's telemetry.
* WiVRn server (Guillaume Meunier, Patrick Nicolas et al): streaming tool for virtual and mixed reality headsets.

### Application specific fixes

Some applications may require additional work. See if it's the case below.

#### Arduino IDE v2

It's necessary to add your user to the dialout group to be able to properly connect to the microcontroller's port:
```
sudo usermod -a -G dialout $USER
```

#### Discord

Go to "appearance" and select the option to change theme accordingly to the system. On system, disable minimize to system tray.

Discord won't allow resizing the window at half screen when the monitor resolution is below 1920x1080. To solve that, open the file `/home/linux/.var/app/com.discordapp.Discord/config/discord/settings.json` and add the lines `"MIN_WIDTH": 0,` and `"MIN_HEIGHT": 0,` before the end of the file.

On old computers, hardware acceleration may introduce problems. Disable it on Discord's "System" options if you notice slowdowns or crashes.

#### Firefox

Old computers may be uncapable of decoding VP8/VP9 videos. The extension h264ify (https://addons.mozilla.org/pt-BR/firefox/addon/h264ify/) allows streaming of h264 videos. It may be useful for computers around 10 or more years old, reducing processor usage.

Computers with less processing power may also struggle to load web pages with too many ads. You can use the uBlock Origin extension (https://addons.mozilla.org/pt-BR/firefox/addon/ublock-origin/) to block ads. Remmeber that blocking ads prevents content creators from getting paid by ad networks. Try to always support your favorite websites and creators!

#### OpenRGB

The udev rules are necessary to have access to RGB devices. They are provided in the "resources" folder within this repository. Open a Console in the same folder you downloaded the file and do:
```
sudo cp 60-openrgb.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

#### Rhythmbox

Rhythmbox is a GTK3 app. The Flatpak version needs the dark theme package and an adjustment on Flatseal to activate dark mode. This, however, will make it only work on dark mode. First, do:
```
flatpak install flathub org.gtk.Gtk3theme.adw-gtk3-dark
```
Then, open Flatseal and add this variable for Rhythmbox: `GTK_THEME=adw-gtk3-dark`. Also delete the default "best classificated" playlist.

#### Steam

Steam needs udev rules to properly connect to joysticks. Install them running this command:
```
sudo apt install steam-devices --no-install-recommends
```
Remote Play may be blocked by our firewall. So, let's enable some ports:
```
sudo ufw allow 27031/udp
sudo ufw allow 27036/udp
sudo ufw allow 27036/tcp
sudo ufw allow 27037/tcp
sudo ufw reload
```
MangoHUD can be used as an in-game system resources monitor. On the software store, search for the Freedesktop Platform. There, install MangoHud. To activate it on a Steam game, try these launch options (you can customize what is shown):
```
MANGOHUD=1 MANGOHUD_CONFIG="fps,frametime,cpu_stats,cpu_temp,gpu_stats,gpu_temp,ram,vram" %command%
```
To limit the maximum FPS rate in a game (for this example, 75 FPS), add to MangoHUD:
```
MANGOHUD=1 MANGOHUD_CONFIG="fps_limit=75" %command%
```
Sometimes, the software store may install a MangoHUD version that has a runtime that differs from Steam's. This will prevent MangoHUD from working. To solve, first check the Steam runtime version:
```
flatpak info com.valvesoftware.Steam | grep Runtime
```
The last numbers (for example, 25.08) are the runtime. Then install a correct version of MangoHUD using the terminal application:
```
flatpak install org.freedesktop.Platform.VulkanLayer.MangoHud
```
Choose the number that matches the runtime used by Steam. In the future, it may be necessary to do this procedure again after Steam updates its runtime.

#### Steam Link

To use joysticks, it is necessary to install the steam-devices package as described above. But also to open Flatseal and add access to the folder /dev/uinput:ro.

#### Unity Hub

Some versions of the Unity Editor crash when loading a project. It is necessary to add a file. Search for the "Data" folder on your Editor install folder (generally located in /home/$USER/Unity). Then rename the original "bee_backend" file as "bee_backend_real" and copy the "bee_backend" file provided in the "resources" folder within this repository. It should be marked as executable.

#### VSCodium

Go to settings, window and "auto detect color scheme" to enable light and dark theme switching.

#### WiVRn

Don't use SteamVR and WiVRn at the same time. WiVRn is a complete replacement for SteamVR. Install the WiVRn server from the GNOME Software Store and the WiVRn client on the Meta Quest, using the Meta Store. The versions must match.

First, open necessary ports on UFW firewall: `sudo ufw allow 5353/udp` and `sudo ufw allow 9757`. Then, allow the Steam Flatpak to access these files (the files access permission can also be granted using Flatseal):
```
flatpak override \
  --filesystem=xdg-run/wivrn:ro \
  --filesystem=xdg-data/flatpak/app/io.github.wivrn.wivrn:ro \
  --filesystem=/var/lib/flatpak/app/io.github.wivrn.wivrn:ro \
  --filesystem=xdg-config/openxr:ro \
  --filesystem=xdg-config/openvr:ro \
  com.valvesoftware.Steam
```
For each Steam game you want to run, add these launch options on Steam:
```
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 PRESSURE_VESSEL_FILESYSTEMS_RW=/var/lib/flatpak/app/io.github.wivrn.wivrn %command%
```
Open the WiVRn server on your computer and it will appear on the headset's client. If necessary (it may not be automatic), change the audio source on the computer for the one created by WiVRn.

## Configuration

### System

- **System:** on screen tab, configure screen resolution and frequency (activate variable refresh rate if available) and activate night light (reducing the intensity to the first level). If you have more than one screen, move the numbered windows until they are in a great position. On energy tab, disable automatic screen turnoff, disable automatic suspension when connected to an outlet, and activate battery percentage show. On multitasking, disable the active corner and choose to show applications only from the current workspace. On appearance, change the wallpaper. On mouse and touchpad, disable mouse acceleration and configure sensibility as wanted. On system, activate the option to show the week day and change the name and picture of your user.
- **General:** on the show applications view, sort your apps by alphabetical order. At the upper menu, click the clock, look for meteorology and choose your city.
- **Adjustments:** on the Adjustments application, choose 0,90 as font scaling. Go to the "Windows" tab and activate maximize and minimize buttons.
- **Dash to Dock:** position and size: disable autohide, choose size 36. Launchers: disable the options to show volumns and the recycling bin. Behavior: change the click action to "minimize or show previews", choose "alternate workspace" as rolling action. Appearance: activate the compact dock option, disable the option to show general view at boot, choose points as the window counting indicators with dominant color, change the dock color to black and fix opacity on 80%.
- **GNOME Disks:** open the Disks application and format any additional drives with ext4, choosing an easy to remember label. Edit mount options: disable user defaults and enable "LABEL" as the identifier, so the disk will be automatically mounted and appear on Nautilus (the file explorer) with its label. You can also choose to "edit filesystem" of any partition and add or change a label.
- **Software:** disable automatic Flatpak updates, disable automatic updates notifications.

### Applications

- **GUFW:** activate the ufw firewall.
- **Nautilus:** activate the option to show folders before files.
- **Firefox:** disable favorites bar, activate the option to always ask where to save downloaded files, disable paid shortcuts, choose DuckDuckGo as the search engine and disable the others except Google. On the privacy and security tab, activate "Tell websites not to sell or share my data" and disable all the Mozilla's telemetry options.
- **Rhythmbox:** on Flatseal, add access permissions for where your songs are saved. Also import your playlists.
- **Mission Center:** on the CPU tab, right click and select "show logical processors".

## Virtual machines

### Debian and Ubuntu based systems

You need to install drivers to have faster speed, automatic resolution resizing and folder sharing on Boxes:
```
sudo apt install spice-vdagent
sudo apt install spice-webdavd
```
It's also good to go to the energy options and disable automatic suspension and screen turnoff.

### Windows 11

To install Windows 11 on Boxes, first we need to disable the TPM 2.0 requirement. Press Shift + F10 when the incompatibility screen appears. Then type `regedit` and go to HKEY_LOCAL_MACHINE\SYSTEM\Setup. Create a new key labeled "LabConfig". Create a 32 bit DWORD value containing `BypassTPMCheck = 1`. Click to go back and try again.

To disable the Microsoft account requirement and create a local account, press Shift + F10 when the region selection screen appears. If you're connected to the Internet using a cable, disconnect it. If it's Wi-Fi, disable it temporarily. Then type `OOBE\BYPASSNRO`. Wait for the system to reboot and procced to the creation of a local account.

To have faster speed and automatic resolution resizing, use your virtual machine to go to https://www.spice-space.org/download.html and download and install the Spice Guest Tools for Windows. To have folder sharing, also install the Spice WebDAV Daemon.

On Windows 11, disable Delivery Optimization. Then go to the privacy options tab and disable every telemetry you can. Also do a little debloat editing your system bar and removing unwanted applications. Finally, start the Disk Cleaning tool, choose to clean system files, select everything and wait.

## Maintenance

### Updating and cleaning

Run the following commands in sequence once a month to update and clean your system:

```
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt clean
sudo apt update
flatpak update
flatpak uninstall --unused
```

## Troubleshooting

### Steam game adjustments

Some Steam games may require additional steps to work or accept mods. See the list below.

#### Battlefield 4

The game has issues if the FPS rate is too high. Since there is no way to limit the FPS in game, use MangoHUD as described above.

#### Counter-Strike 2

The game may fail to capture mouse inputs. On Steam, insert `SDL_VIDEO_DRIVER=x11 %command%` as a launch option.

#### Cyberpunk (mods)

Open the game at least one time. Then, install the "Protontricks" application and access the game's prefix. Select the option to install Windows components. Choose d3dcompiler_47 and vcrun2022, install and close. On Steam, insert `WINEDLLOVERRIDES="winmm,version=n,b" %command%` as a launch option.

#### Rocket League

The Linux version was discontinued after Epic Games unfortunately bought the game. However, you can force Steam to use Proton Experimental and use the Windows version normally.

#### Rocksmith 2014 Remastered (Real Tone Cable)

Open the game at least one time. Then, install the "Protontricks" application and access the game's prefix. Choose the option to change configurations. Choose `sound=alsa` as the sound engine. Now go back on Protontricks and "execute winecfg". Select the Real Tone Cable as the entry audio source. Now change these options on the Rocksmith.ini file (located at the game's install folder):
```
EnableMicrophone=1
ExclusiveMode=0
LatencyBuffer=1
ForceDefaultPlaybackDevice=0
ForceWDM=0
ForceDirectXSink=0
DumpAudioLog=0
MaxOutputBufferSize=1024
RealToneCableOnly=1
MonoToStereoChannel=0
Win32UltraLowLatencyMode=0
```

#### The Elder Scrolls Online (AddOns)

Minion is available to install at the Software application to manage AddOns. The folder's default location is `/home/arch/.var/app/com.valvesoftware.Steam/.steam/steam/steamapps/compatdata/306130/pfx/drive_c/users/steamuser/Documents/Elder Scrolls Online/live/AddOns/`.

I also have an application to update Tamriel Trade Centre's (TTC) database. Check the tool's repository: https://github.com/kairand/ttc-eso-linux