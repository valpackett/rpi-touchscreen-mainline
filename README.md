# Raspberry Pi Official 7" Touchscreen on Mainline Linux Kernel

The mainline kernel includes a panel driver for the 7" touchscreen.
[A device tree diff to enable it](https://lists.freedesktop.org/archives/dri-devel/2017-June/144443.html) exists.
An overlay was planned, but it's still nowhere to be found.
Plus, the touch input and backlight drivers are not included in the mainline kernel.

Turns out, it's really easy to compile the input/backlight drivers out of tree!
They don't depend on anything that's not upstreamed.

This repo includes copies of these modules and a device tree.

## Kernel Modules

### Installation

```bash
make modules
sudo make install
```

### Loading

```
sudo insmod /lib/modules/$(uname -r)/extra/rpi-ft5406.ko.gz
sudo insmod /lib/modules/$(uname -r)/extra/rpi_backlight.ko.gz
```

## Flattened Device Tree

### Recompilation

```bash
dtc -I dts -O dtb -o upstream.dtb upstream.dts
```

**The included device tree is for the Pi 3 B**, if you're on 2 or whatever, you'll need to make your own:

- Decompile the upstream dtb for your Pi (`dtc -I dtb -O dts` ...)
- Add labels:
  - `firmware {` -> `firmware: firmware {`
  - `gpio@7e200000 {` -> `gpio: gpio@7e200000 {`
  - `dsi@7e700000 {` -> `dsi1: dsi@7e700000 {`
- Add the `i2c_dsi: i2c` section and modify the `dsi1: dsi@7e700000` section (don't forget to remove `status = "disabled";`) like in [that diff](https://lists.freedesktop.org/archives/dri-devel/2017-June/144443.html), using appropriate gpio numbers (they're shown there as `&i2c_dsi` blocks for various Pis' separate files, just take the gpio numbers and modify them inline in the `i2c_dsi: i2c` section)
- Add the `rpi_ft5406` and `rpi_backlight` sections from my `upstream.dts`
- Finally, compile!

Look at the diff linked above, it has different numbers for different Pis.

### Usage

Depends on your distro and way of booting.

For netbooting from U-Boot, something like this:

```
tftp ${kernel_addr_r} /Image
tftp ${fdt_addr_r} /upstream.dtb
tftp ${ramdisk_addr_r} /initramfs-linux.img
booti ${kernel_addr_r} ${ramdisk_addr_r}:${filesize} ${fdt_addr_r}
```

For loading from the firmware, in `config.txt`:

```
device_tree=upstream.dtb
```
