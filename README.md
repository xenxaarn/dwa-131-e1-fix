# WiFi Driver Fix for rtl8192eu (D-Link adapter)

Works on **all major Linux distros**: Debian/Ubuntu/Pop!_OS, Fedora/RHEL, Arch/Manjaro, openSUSE, Gentoo.

## Step 1: Install Build Tools and Dependencies

Pick the block for your distro:

### Debian / Ubuntu / Pop!_OS

```bash
sudo apt update
sudo apt install git linux-headers-generic build-essential dkms
```

### Fedora / RHEL / Rocky / AlmaLinux

```bash
sudo dnf install git kernel-devel gcc make dkms
```

> If `kernel-devel` fails, use: `sudo dnf install kernel-devel-$(uname -r)`

### Arch / Manjaro / EndeavourOS

```bash
sudo pacman -Syu
sudo pacman -S git base-devel dkms linux-headers
```

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
rm -f /etc/modprobe.d/rtlxxxu.conf
echo "blacklist rtl8xxxu" | sudo tee /etc/modprobe.d/rtl8xxxu.conf
```

## Step 4: Register and Install via DKMS

This ensures the driver builds automatically whenever the system updates the kernel:

```bash
sudo dkms add .
sudo dkms install rtl8192eu/1.0
```

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
echo -e "8192eu\n\nloop" | sudo tee /etc/modules
echo "options 8192eu rtw_power_mgnt=0 rtw_enusbss=0" | sudo tee /etc/modprobe.d/8192eu.conf
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
sudo systemctl reboot -i
```

Once rebooted, your D-Link adapter will be fully operational, and you won't have to worry about system updates breaking it anymore!
