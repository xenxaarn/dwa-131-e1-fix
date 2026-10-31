# WiFi Driver Fix for rtl8192eu (D-Link adapter)

Works on **all major Linux distros**: Debian/Ubuntu/Pop!_OS, Fedora/RHEL, Arch/Manjaro, openSUSE, Gentoo.

## Step 1: Install Build Tools and Dependencies

Pick the block for your distro:

### Debian / Ubuntu / Pop!_OS

```bash
sudo apt update
sudo apt install git linux-headers-$(uname -r) build-essential dkms
```

### Fedora / RHEL / Rocky / AlmaLinux

```bash
sudo dnf install git kernel-devel-$(uname -r) gcc make dkms
```

> If that exact version isn't found, run `sudo dnf upgrade`, reboot, and try again.

### Arch / Manjaro / EndeavourOS

```bash
sudo pacman -Syu
sudo pacman -S git base-devel dkms linux-headers
```

> If the upgrade installed a new kernel, reboot before continuing. Non-default kernels need their own headers (e.g. `linux-lts-headers`).

### openSUSE

```bash
sudo zypper install git kernel-devel gcc make dkms
```

### Gentoo

```bash
sudo emerge --ask dev-vcs/git sys-kernel/dkms sys-kernel/linux-headers
```

## Step 2: Clone the Open-Source Driver

```bash
git clone https://github.com/Mange/rtl8192eu-linux-driver.git
cd rtl8192eu-linux-driver
```

## Step 3: Blacklist the Broken Native Driver

Stop the distro from using the buggy built-in `rtl8xxxu` kernel module (remove the wrong blacklist if it was created before):

```bash
sudo rm -f /etc/modprobe.d/rtlxxxu.conf
echo "blacklist rtl8xxxu" | sudo tee /etc/modprobe.d/rtl8xxxu.conf
```

## Step 4: Register and Install via DKMS

This ensures the driver builds automatically whenever the system updates the kernel:

```bash
sudo dkms add .
sudo dkms install rtl8192eu/1.0
```

### Secure Boot (only if enabled)

```bash
mokutil --sb-state
```

If it says `SecureBoot enabled`, the kernel rejects unsigned modules (`Key was rejected by service`). DKMS signs the module with its own key, which you must enroll once:

```bash
ls /var/lib/dkms/mok.pub || sudo dkms generate_mok
sudo mokutil --import /var/lib/dkms/mok.pub
sudo reboot
```

At the blue MOK Manager screen choose *Enroll MOK → Continue → Yes*, enter the password you set, and reboot. If `/var/lib/dkms/mok.pub` isn't your key location, check `mok_certificate` in `/etc/dkms/framework.conf`. Or disable Secure Boot in firmware.

## Step 5: Load the Driver for the Current Kernel

```bash
KERN=$(uname -r)
sudo dkms build rtl8192eu/1.0 -k "$KERN"
sudo dkms install rtl8192eu/1.0 -k "$KERN"
sudo modprobe -r rtl8xxxu 2>/dev/null || true
sudo modprobe 8192eu
```

## Step 6: Configure Power & Boot Rules

Prevent idling drops and force the driver to initialize correctly on boot:

```bash
echo -e "8192eu\n" | sudo tee /etc/modules-load.d/8192eu.conf
echo 'omit_dracutmodules+=" rtl8xxxu "' | sudo tee /etc/dracut.conf.d/blacklist-rtl8xxxu.conf
```

Then rebuild the initramfs, pick the block for your distro:

### Debian / Ubuntu / Pop!_OS

```bash
sudo update-initramfs -u
```

### Fedora / RHEL / Rocky / AlmaLinux

```bash
sudo dracut --force
```

### Arch / Manjaro / EndeavourOS

```bash
sudo mkinitcpio -P
```

### openSUSE

```bash
sudo mkinitrd
```

### Gentoo

```bash
sudo dracut --force
```

## Step 7: Verify

```bash
dkms status | grep 8192eu
lsmod | grep -E "8192eu|rtl8"
ip link show | grep -i wl
```

## Step 8: Reboot Your System

```bash
sudo systemctl reboot
```

Once rebooted, your D-Link adapter will be fully operational, and DKMS will rebuild the driver on kernel updates (as long as the matching kernel headers are installed).
