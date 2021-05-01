# Fedora 33 Setup with an ASUS Zephyrus G14 2020 / 2021 model

---

This Git repo describes how I setup the ASUS Zephyrus G14 (GA401IV) with Fedora 33 including a GNOME Shell extension to switch between GPUs and ROG profiles.

This also should work with the 2021 models as the new custom kernel is in here. Perhaps with them you need to boot the live media with `nomodeset` and remove this parameter after you installed the nvidia-driver.

## Installation process

**1. Install Fedora 33**

**2. Boot to your installation, add Lukes Copr and update everything**

```bash
dnf copr enable lukenukem/asus-linux
dnf update --refresh
dnf install asusctl
```

> this also will install a custom kernel that helps with the 2021 G14 model to get suspend working

> this also will fix the touchpad issues with all the G14 models that don't initialize correctly sometimes

> asusctl helps with managing ROG power profiles and switching between graphics modes

**3. Reboot**
```bash
reboot
```

**4. copy all the files to the appropriate directories**

```bash
git clone https://github.com/hyphone/asus-g14-fedora.git
cd asus-g14-fedora
cp -R etc/* /etc/
cp -R usr/* /usr/
systemd-hwdb update
udevadm trigger
chmod +x /usr/bin/asus_backlight_off
systemctl enable asus_backlight_off_after_suspend.service
systemctl enable asus_backlight_off_before_suspend.service
systemctl enable asus_hibernate.service
```
> we clone this repo

> we go to the repo directory

> we copy everything in this repo of etc to /etc/

> we copy everything in this repo of usr to /usr/

> mod the keyboard that **page up / down** is mapped to **fn+up/down** while **home (pos1) / end** is mapped to **fn+left/right**

> you can use `brightnessctl -d asus::kbd_backlight s +1` and `brightnessctl -d asus::kbd_backlight s 1-` and map this to a key of your choice in your desktop environment

> `asus_backlight_off_after_suspend.service` / `asus_backlight_off_before_suspend.service` will turn off the keyboard backlight before and after suspend.<br>There is a bug that the keyboard backlight does not fully turn off on it's own when suspending when it was on before. After suspend the backlight is usually in a different state than before. To unify this behaviour we just turn it off when suspending and on waking the machine.<br>I made services out of it and haven't put it in /usr/lib/systemd/system-sleep because of an SELinux bug that prevents scripts from executing there on F34.

> asus_hibernate.service restarts asusd after hibernation to re-apply the power profile.


**5. install some packages**
```bash
dnf install --refresh kernel-devel akmod-nvidia xorg-x11-drv-nvidia-cuda akmod-acpi_call brightnessctl
```

> akmod-nvidia and xorg-x11-drv-nvidia-cuda installs the Nvidia driver

> kernel-devel is necesarry for the dynamic kernel modules to compile

> acpi_call modules from the tlp repo is needed to make the custom fan control working

> brightnessctl is used for controlling the keyboard backlight before and after suspend as there is a bug that the keyboard backlight sometimes does not switch completely off while suspending. I also use it to control the brightness with the keyboard as I'm overriding the default keys with page up / down.

**6. Reboot**
```bash
reboot
```

**7. You can switch your prefered graphics mode via the GNOME Shell extension "asusctl-gex" or with "asusctl graphics -m (graphics mode)"**

---

## What's in here...

### asusctl config with custom fan curves

There is an example in etc/asusd/asusd_example.conf for how a custom fan curve should look like.
I haven't named it asusd.conf so it does not override the defaults from asusctl and asusd.

Custom fan curves have a problem at least on the 2020 models that sometimes the fans start to pulsate and one has to suspend one or two times to bring them back to a normal state.

I also currently don't use custom fan curves because of this.

### Asus-nb-gex GNOME Shell extension (currently in development)

The main repo for the Gnome Shell extension can be found here: ![https://gitlab.com/asus-linux/asus-nb-gex](https://gitlab.com/asus-linux/asus-nb-gex)
It is currently still in development when we find the time. In this repo here you find the most current version that I also use:
![https://gitlab.com/asus-linux/asus-nb-gex/-/tree/dev/1.0.0](dev/1.0.0).

![](https://raw.githubusercontent.com/hyphone/asus-g14-fedora/master/screenshot.png)

**Graphics Mode**

- _integrated_ - only use integrated AMD GPU while the NVIDIA is completely turned off
- _hybrid_ - use the integrated AMD GPU as the main GPU. The NVIDIA can be used for offloading *1
- _compute_ - use the integrated AMD GPU as the main GPU. The NVIDIA can be used for CUDA / OpenCL but not for offloading graphics.
- _vfio_ - use the integrated AMD GPU as the main GPU. The NVIDIA can be used in combination with a VM for GPU passthrough.
- _nvidia_ - use the dedicated NVIDIA GPU as the main GPU.

*1 to offload a graphics application to your NVIDIA you can use GNOME Shells feature via right click on an application icon and use **run with dedicated graphics adapter**

**Profile**
- _Boost_ - high fan RPM, Ryzen Boost enabled
- _Normal_ - silent fan until 49°C CPU temperature, spins up at higher temps, Ryzen Boost enabled
- _Silent_ - silent fan until 69°C CPU temperature, spins up at higher temps, Ryzen Boost disabled

**usually a reboot or a restart of Xorg is required to apply the changes. There will be a notification with a call to action to give you a hint what action is required.**
![](https://user-images.githubusercontent.com/6410852/106669281-b8d96500-65ab-11eb-9e1e-adbd0126587d.png)
_**please note** that the message sometimes is not correct when you switch multiple times without doing the appropriate action. So save your work and do as the asusctl and this extension tells you :)_

> **silent** is good for the **integrated** graphics mode. for **every other** graphics mode I recommend **normal** as the case would heat up too much on silent.

> This extension uses asusctl to switch modes via asusctl's DBUS interface. You can either use the extension or asusctl directly if you don't use GNOME Shell.

> If you don't have GNOME Shell but a desktop environment with a system tray I suggest you use this very handy application: ![asusctltray](https://github.com/Baldomo/asusctltray/)

### asusctl

![asusctl](https://gitlab.com/asus-linux/asusctl) by Luke Jones (and his Kernel modules) are the key to this all. Without the efford of the community we wouldn't have such handy tools to control this machine. asusctl is used here to do most of the things, like graphics switching via the GNOME Shell extension or setting the ROG profiles.

You can use asusctl to switch your profile like so (use _integrated_, _hybrid_, _compute_ or _nvidia_ instead of {profile}):
```
asusctl graphics -m {profile}
```

You can also change the ROG profile via asusctl (use _silent_, _normal_ or _boost_ instead of {profile}):
```
asusctl profile {profile}
```

for more options have a look at this command:
```
asusctl --help
```

***

```
etc/asusd/asusd_example.conf
```
These are my custom fan curves for the asusd service from asus-linux.org.
When on AC I usually use "normal" because of turbo is enabled with the default silent fan profile of the laptop.
On "silent" I disabled turbo and I usually use this while on battery. This has a custom fan curve to make it silent.


```
etc/modules-load.d/...
```
- acpi_call.conf makes sure the acpi_call module is loaded (when installed) to get custom fan control working