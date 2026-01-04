# Troubleshooting MT7921e Driver Errors on Linux

If your kernel logs are showing the following errors:

```
mt7921e ... driver own failed
probe with driver mt7921e failed with error -5
```

And you are having problems with WiFi, this guide should fix your issues.

## Prerequisites

Before trying any of the fixes bellow, check if your system meets the baseline requirements for Mediatek cards.

### Check kernel version

Run `uname -r` - if your kernel version is <6.0, you should probably update your kernel. Mediatek WiFi cards have problems with older kernels.
If you have kernel version 6.1+, proceed with this guide.

### Check for firmware

The driver requires specific binary blobs to function. Check if they exist by running `ls /lib/firmware/mediatek | grep mt7921`. You should find at least the following files:

```
mt7921_rom_patch.bin
mt7921_wa.bin
```

**If these files are missing, continue to the fixes below.** If they are present, the problem lies somewhere else.

## Fix: Restoring the missing firmware

### Method A: Use linux-firmware

If the firmware files are missing, the easiest fix is to try to update or (re)install `linux-firmware`. Do this with your distros package manager, and reboot after:

#### Debian & friends

``` bash
sudo apt update
sudo apt install linux-firmware # for reinstallation add --reinstall
sudo reboot
```

#### Fedora/RHEL & friends

``` bash
sudo dnf makecache --refresh
sudo dnf install linux-firmware # for reinstallation replace "install" with "reinstall"
sudo reboot
```

#### Arch & friends

``` bash
sudo pacman -Sy linux-firmware # same command for installation and reinstallation
sudo reboot
```

If the files are still missing, move to Method B.

## Method B: Manual installation

Some distros (like Fedora) don't ship the firmware files, so you need to download them by yourself.

``` bash
# Download to a temporary folder
cd /tmp
curl -LO https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/mediatek/mt7921_rom_patch.bin
curl -LO https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/mediatek/mt7921_wa.bin
```

Then copy them to the directory that contains the firmware files

``` bash
# Move to the firmware directory
sudo mkdir /lib/firmware/mediatek/ # create if missing
sudo cp mt7921_*.bin /lib/firmware/mediatek/
```

Set the correct permissions

``` bash
# Set standard read permissions
sudo chmod 644 /lib/firmware/mediatek/mt7921_*.bin
```

## Fix: Stability tweaks

Even with the correct firmware MT7921e often crashes due to power management issues.

### Disable ASPM for MT7921e

Create a module configuration file to disable ASPM.

``` bash
sudo tee /etc/modprobe.d/mt7921e.conf <<EOF
options mt7921e disable_aspm=1
EOF
```

### Optional: Disable NetworkManager powersaving

If you're using NetworkManager, you should disable WiFi power saving. This prevents random disconnects / suspend issues.

``` bash
sudo tee /etc/NetworkManager/conf.d/wifi-powersave.conf <<EOF
[connection]
wifi.powersave = 2
EOF
```

Restart NetworkManager after:

``` bash
sudo systemctl restart NetworkManager
```

## Apply changes

Do this for the changes to take effect:

### Using initramfs

If you are using an initramfs, you need to rebuild it for the changes to take effect.

#### Debian & friends

``` bash
sudo update-initramfs -u
sudo reboot
```

#### Fedora/RHEL & friends

``` bash
sudo dracut --force --verbose
sudo reboot
```

#### Arch & friends

``` bash
sudo mkinitcpio -P
sudo reboot
```

This should make the card work and stop throwing errors mentioned at the start to the kernel log.

### Without initramfs

If you need to, recompile your kernel to include the new firmware files. Then reboot your system.

## Verified systems

I, myself, tested the guide on:

- Fedora 43,
- Kernel version 6.17.12-300.fc43.x86_64,
- NetworkManager version 1.54.3-2.fc43.

If anyone tests the guide on some other system, and reports success, please contact me or make a PR and I will add your system configuration to the list.
