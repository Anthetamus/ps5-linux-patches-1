# ps5-linux-patches

Linux patches and kernel config for the PS5.
This is a fork of [ps5-linux/ps5-linux-patches](https://github.com/ps5-linux/ps5-linux-patches).
The patch targets mainline 7.3.

## Build with build.sh

Run this on the PS5.

### What you need

Arch:

```bash
sudo pacman -S --needed base-devel git bc cpio libelf clang lld llvm ccache mkinitcpio
```

Ubuntu:

```bash
sudo apt install build-essential bc bison flex libssl-dev libelf-dev libncurses-dev dwarves git clang lld llvm ccache
```

`ccache` is only needed with `CCACHE=y` (the default).

### Steps

```bash
git clone https://github.com/rmuxnet/ps5-linux-patches-1 ps5-linux-patches
cd ps5-linux-patches
./build.sh
```

The script clones the latest mainline tag into `../linux`, applies the patch and builds with clang.
It installs the modules, backs up the boot partition to `/boot/efi-backup/<date>` and copies the new kernel and initrd there.
The script looks for a mounted partition with `bzImage` on it and asks if it is the right one.
Press Enter or `y` to use it. Press `n` to type the path yourself.
To skip the question, set `EFI_DIR` in `build.cfg`.
Cold boot the PS5 after that.

The tree in `../linux` stays between runs, so the next build is faster.

### Options

All options are in `build.cfg`. To use your own file:

```bash
cp build.cfg my.cfg
./build.sh my.cfg
```

### Wi-Fi

The Wi-Fi driver is out of tree.
Kernel 7.3 needs a compat patch on top of [ps5-linux-mwifiex](https://github.com/ps5-linux/ps5-linux-mwifiex).
Prepare and patch that tree first. Then set `WIFI_SRC` to it.
If `WIFI_SRC` is empty, rebuild the Wi-Fi modules yourself after each kernel build.

### Roll back

The script prints the backup folder. Copy the old files back:

```bash
sudo cp /boot/efi-backup/<date>/{bzImage,initrd.img} <boot partition>/
sync
```

Then cold boot.

## Build by hand

```bash
git clone https://github.com/rmuxnet/ps5-linux-patches-1 ps5-linux-patches
git clone --depth 1 --branch v7.3-rc6 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git apply ../ps5-linux-patches/linux.patch
cp ../ps5-linux-patches/.config .config
make olddefconfig
make -j$(nproc)
sudo make INSTALL_MOD_STRIP=1 modules_install
```

Use the tag from the first lines of `.config`.
Keep the folder name `ps5-linux-patches`. The `.config` loads the VCN firmware from `../ps5-linux-patches/firmware`.

`make install` does not copy the kernel on the PS5. Copy it to the boot partition and make an initrd.
This example uses `/boot/efi`:

```bash
KVER=$(make -s kernelrelease)
sudo cp arch/x86/boot/bzImage /boot/efi/bzImage
sudo mkinitcpio -k $KVER -g /boot/efi/initrd.img
```

Back up the old `bzImage` and `initrd.img` before you do this.

## TODO

- amdgpu smu driver to show correct gpu frequency and temperature
- hdmi converter improvments: hdr, rgb range, 120hz

## Bugs

- screen save does not work properly
- hdmi audio output does not work on some monitors
- hdmi 1440p and 2160p video output does not work on some monitors
- monitor swap is not supported
- hdmi cec is not supported
