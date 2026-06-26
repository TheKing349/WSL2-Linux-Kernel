# Controller Support

This is a fork of the [Main WSL2 Kernel](https://github.com/microsoft/WSL2-Linux-Kernel) that has been modified for USB gamepad support. This is based off of another [fork](https://github.com/atticusrussell/WSL2-Linux-Kernel) of the kernel, but is out of date. This serves as an updated and maintained project following releases of the main repository.

## Modifications

There are not a lot of changes needed, just an updated Makefile. The `Microsoft/config-wsl` serves as a Makefile, in which I modified a few lines (check out this [commit](https://github.com/TheKing349/WSL2-Linux-Kernel/commit/9243fd45280f85c4cc38c729c2dbf2d98a31d940) to learn more).

From there, I recompiled the kernel as directed [here](#build-instructions), which resulted in the binary of `vmlinux` and a `modules.vhdx`, both of which are needed for this custom kernel.

Lastly, I verified everything worked correctly by loading the custom kernel using the [Microsoft Documentation](https://learn.microsoft.com/en-us/windows/wsl/wsl-config) to use the custom binaries. Then I used [usbipd](https://learn.microsoft.com/en-us/windows/wsl/connect-usb) to connect a controller, and did `sudo evtest` to ensure functionality.

To use this for yourself, head over to the [releases](https://github.com/TheKing349/WSL2-Linux-Kernel/releases/latest) page and download the `vmlinux+modules.zip` file. Unzip, and link them to your `.wslconfig`.

--

# Original README

## Introduction

The [WSL2-Linux-Kernel][wsl2-kernel] repo contains the kernel source code and
configuration files for the [WSL2][about-wsl2] kernel.

## Reporting Bugs

If you discover an issue relating to WSL or the WSL2 kernel, please report it on
the [WSL GitHub project][wsl-issue]. It is not possible to report issues on the
[WSL2-Linux-Kernel][wsl2-kernel] project.

If you're able to determine that the bug is present in the upstream Linux
kernel, you may want to work directly with the upstream developers. Please note
that there are separate processes for reporting a [normal bug][normal-bug] and
a [security bug][security-bug].

## Feature Requests

Is there a missing feature that you'd like to see? Please request it on the
[WSL GitHub project][wsl-issue].

If you're able and interested in contributing kernel code for your feature
request, we encourage you to [submit the change upstream][submit-patch].

## Build Instructions

Instructions for building an x86_64 WSL2 kernel with an Ubuntu distribution using bash are
as follows:

1. Install the build dependencies:  
   `$ sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev cpio qemu-utils`

2. Modify WSL2 kernel configs (optional):  
   `$ make menuconfig KCONFIG_CONFIG=Microsoft/config-wsl`

3. Build the kernel using the WSL2 kernel configuration and put the modules in a `modules`
   folder under the current working directory:  
   `$ make KCONFIG_CONFIG=Microsoft/config-wsl && make INSTALL_MOD_PATH="$PWD/modules" modules_install`
   
   You may wish to include `-j$(nproc)` on the first `make` command to build in parallel.

Then, you can use a provided script to create a VHDX containing the modules:
   `$ sudo ./Microsoft/scripts/gen_modules_vhdx.sh "$PWD/modules" $(make -s kernelrelease) modules.vhdx`

To save space, you can now delete the compilation artifacts:
   `$ make clean && rm -r "$PWD/modules"`

If you prefer, you can also build the modules VHDX manually as follows:

1. Calculate the modules size (plus 256MiB for slack):
   `modules_size=$(du -bs "$PWD/modules" | awk '{print $1;}'); modules_size=$((modules_size + (256 * (1<<20))));`

2. Create a blank image file for the modules:
   `dd if=/dev/zero of="$PWD/modules.img" bs=1024 count=$((modules_size / 1024))`

3. Setup filesystem and mount img file:
   `lo_dev=$(sudo losetup --find --show "$PWD/modules.img") && sudo mkfs -t ext4 "$lo_dev" && mkdir "$PWD/modules_img" && sudo mount "$lo_dev" "$PWD/modules_img"`

4. Copy over the modules, unmount the img now that we're done with it:
   `sudo cp -r "$PWD/modules/lib/modules/$(make -s kernelrelease)"/* "$PWD/modules_img" && sudo umount "$PWD/modules_img"`

5. Convert the img to VHDX:
   `qemu-img convert -O vhdx "$PWD/modules.img" "$PWD/modules.vhdx"`

6. Clean up:
   `rm modules.img # optionally $PWD/modules dir and the now-empty $PWD_modules_img dir too`

## Install Instructions

Please see the documentation on the [.wslconfig configuration
file][install-inst] for information on using a custom built kernel.

[wsl2-kernel]:  https://github.com/microsoft/WSL2-Linux-Kernel
[about-wsl2]:   https://docs.microsoft.com/en-us/windows/wsl/about#what-is-wsl-2
[wsl-issue]:    https://github.com/microsoft/WSL/issues/new/choose
[normal-bug]:   https://www.kernel.org/doc/html/latest/admin-guide/bug-hunting.html#reporting-the-bug
[security-bug]: https://www.kernel.org/doc/html/latest/admin-guide/security-bugs.html
[submit-patch]: https://www.kernel.org/doc/html/latest/process/submitting-patches.html
[install-inst]: https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configure-global-options-with-wslconfig
