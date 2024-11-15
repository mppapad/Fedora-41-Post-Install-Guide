# Fedora 41 Post Install Guide
Things I do after installing Fedora 41
## RPM Fusion
* If you forgot to enable third party repositories during the initial setup window, enable them by pasting the following into the terminal: 
 ```
 sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
 ```
* to install app-stream metadata paste:
 ```
 sudo dnf update @core
 ```
## Update 
* Go into the software center and click on update. Alternatively, you can use the following commands:
```
sudo dnf -y update
```
* To make sure all packages are updated run:
```
sudo dnf -y upgrade --refresh
```
* Reboot
## Firmware
* If your system supports firmware update delivery through lvfs, update your device firmware by:
```
sudo fwupdmgr refresh --force
sudo fwupdmgr get-devices # Lists devices with available updates.
sudo fwupdmgr get-updates # Fetches list of available updates.
sudo fwupdmgr update
```
## Flatpak
* Fedora doesn't include all non-free flatpaks by default. The command below enables access to all the flathub flatpaks. Particularly useful for users of Fedora KDE and other spins since they do not get the "Enable Third Party Repositories" option on initial boot.
```
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```
## NVIDIA Drivers
* Only follow this if you have a NVIDIA gpu. Also, don't follow this if you have a gpu which has dropped support for newer driver releases i.e. anything earlier than nvidia GT/GTX 600, 700, 800, 900, 1000, 1600 and RTX 2000, 3000, 4000 series. Fedora comes preinstalled with NOUVEAU drivers which may or may not work better on those remaining older GPUs. This should be followed by Desktop and Laptop users alike.
* Disable Secure Boot.
* `sudo dnf update` #To make sure you're on the latest kernel and then reboot.
* Enable RPM Fusion Nvidia non-free repository in the app store and install it from there,
* or alternatively
* `sudo dnf install akmod-nvidia`
* Install this if you use applications that can utilise CUDA i.e. Davinci Resolve, Blender etc.
* `sudo dnf install xorg-x11-drv-nvidia-cuda`
* Wait for atleast 5 mins before rebooting, to let the kermel module get built.
* `modinfo -F version nvidia` #Check if the kernel module is built.
* Reboot
## Media Codecs
* Install these to get proper multimedia playback.
````
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing # Switch to full FFMPEG.
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin # Installs gstreamer components. Required if you use Gnome Videos and other dependent applications.
````
## H/W Video Acceleration
* Helps decrease load on the CPU when watching videos online by alloting the rendering to the dGPU/iGPU. Quite helpful in increasing battery backup on laptops.
### H/W Video Decoding with VA-API 
* `sudo dnf install ffmpeg ffmpeg-libs libva libva-utils`
<details>
<summary>Intel</summary>
 
* If you have a recent Intel chipset (5th Gen and above) after installing the packages above., Do:
```
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing
```
</details>
<details>
<summary>AMD</summary>No need to do this for intel integrated graphics. Mesa drivers are for AMD graphics, who lost support for h264/h245 in the fedora repositories in f38 due to legal concerns.
 
* If you have an AMD chipset, after installing the packages above do:
```
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```
* If using i686 compat libraries (for steam or alikes): 
```
sudo dnf swap mesa-va-drivers.i686 mesa-va-drivers-freeworld.i686
sudo dnf swap mesa-vdpau-drivers.i686 mesa-vdpau-drivers-freeworld.i686
```
</details>
<details>
<summary>NVIDIA</summary>
 
* The Nvidia proprietary driver doesn't support VAAPI natively, but there is a wrapper that can bridge NVDEC/NVENC with VAAPI, Use:
```
sudo dnf install libva-nvidia-driver
```
* You can also install both 32bit and 64bit flavor in one command as needed. 
``` 
sudo dnf install libva-nvidia-driver.{i686,x86_64}
```
</details>
### OpenH264 for Firefox
* You can enable the OpenH264 Plugin directly in Firefox's settings.
## Set Hostname
```
hostnamectl set-hostname YOUR_HOSTNAME
```
## Optimizations [Optional]
* The tips below can allow you to squeeze out a little bit more performance from your system. 
### Disable Mitigations 
* Increases performance in multithreaded systems. The more cores you have in your cpu the greater the performance gain. 5-30% performance gain varying upon systems. Do not follow this if you share services and files through your network or are using fedora in a VM. 
* Modern intel CPUs (above 10th gen) do not gain noticeable performance improvements upon disabling mitigations. Hence, disabling mitigations can present some security risks against various attacks, however, it still _might_ increase the CPU performance of your system.
```
sudo grubby --update-kernel=ALL --args="mitigations=off"
```
### Modern Standby [Recommended for Laptops]
* Can result in better battery life when your laptop goes to sleep.
```
sudo grubby --update-kernel=ALL --args="mem_sleep_default=s2idle"
```
* If "s2idle" doesn't work for you i.e. people with alder lake CPUs, then you might want to refer to [this](https://www.reddit.com/r/linuxhardware/comments/ng166t/s3_deep_sleep_not_working/)
### Enable nvidia-modeset 
* Useful if you have a laptop with an Nvidia GPU. Necessary for some PRIME-related interoperability features.
```
sudo grubby --update-kernel=ALL --args="nvidia-drm.modeset=1"
```
### Disable `NetworkManager-wait-online.service`
* Disabling it can decrease the boot time by at least ~15s-20s:
```
sudo systemctl disable NetworkManager-wait-online.service
```
### Disable Gnome Software from Startup Apps
* Gnome software autostarts on boot for some reason, even though it is not required on every boot unless you want it to do updates in the background, this takes at least 100MB of RAM upto 900MB (as reported anecdotically). You can stop it from autostarting by:
```
sudo rm /etc/xdg/autostart/org.gnome.Software.desktop`
```
## Apps [Optional]
* If you are using a compatible Lenovo Legion laptop i suggest installing The LenovoLegionLinux using corp.
* Enable the repository using:
```
dnf copr enable mrduarte/LenovoLegionLinux 
``` 
* Then install the Application using:
```
sudo dnf install python-darkdetect dkms-LenovoLegionLinux
```
## Special thanks 👏
* [RPMFusion HowTo](https://rpmfusion.org/Howto) | For further instructions and commands.
* #### Remember to give a ⭐ to:
	* [devangshekhawat](https://github.com/devangshekhawat/Fedora-40-Post-Install-Guide) | About the original F40 guide on which this guide is based!
	* [johnfanv2](https://github.com/johnfanv2/LenovoLegionLinux) | For the fantastic effort he produced (LenovoLegionLinux)!
