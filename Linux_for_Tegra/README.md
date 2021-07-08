# RT-WiFi kernel Installation Guide on NVIDIA Jetson Nano

Note: This guide only works on compiling and install on a Jetson device, not for cross compiling.

Before you get hands on compiling and install modules, make sure that you have already read the documents provided by NVIDIA.

[Quick Start Guide](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3231/index.html#page/Tegra Linux Driver Package Development Guide/quick_start.html)

[Developer Guide](mercury.pr.erau.edu/~siewerts/cs415/documents/Jetson/Tegra_Linux_Driver_Package_Developers_Guide.pdf) (**Just for reference! Outdated!**)

[Kernel Customization](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra Linux Driver Package Development Guide/kernel_custom.html#)

**Make sure that you have downloaded all files in the *Linux_for_Tegra* folder.**

## Building the NVIDIA kernel

Follow the steps in this procedure to build the NVIDIA kernel.

### Prerequisites

* You have downloaded the kernel source code (In &lt;top&gt;/linux-4.9/Linux_for_Tegra/source/public/kernel/kernel-4.9).

* You have installed the utilities required by the kernel build process.

  Install the utilities with the command:

  ```bash
  $ sudo apt install build-essential bc libncurses5-dev libncursesw5-dev libssl-dev
  ```

### To build the Jetson Linux Kernel

1. Set the shell variable with the command:

   ```bash
   $ TEGRA_KERNEL_OUT=<outdir>
   ```

   Where:

   `<outdir>` is the desired destination for the compiled kernel (Please use full dir, e.g. /home/test/output)

2. If cross-compiling on a non-Jetson system, export the following environment variables:
   ```bash
   $ export CROSS_COMPILE=<cross_prefix>
   $ export LOCALVERSION=-tegra
   ```
   
   Where:

   `<cross_prefix>` is the absolute path of the ARM64 toolchain without the gcc suffix. For example, for the reference ARM64 toolchain, 

   `<cross_prefix>` is:
   `<toolchain_install_path>/bin/aarch64-linux-gnu-`

   See [The L4T Toolchain](https://docs.nvidia.com/jetson/l4t/Tegra%20Linux%20Driver%20Package%20Development%20Guide/xavier_toolchain.html#) for information on how to download and build the reference toolchains.

3. Execute the following commands to create the .config file:

   ```bash
   $ cd <kernel_source>
   $ mkdir -p $
   $ make ARCH=arm64 O=$TEGRA_KERNEL_OUT tegra_defconfig
   ```

   Where:

   `<kernel_source>` directory contains the kernel sources.

4. Execute the following commands to enter the menuconfig and activate the RT-WiFi function:

   ```bash
   make ARCH=arm64 O=$TEGRA_KERNEL_OUT menuconfig
   ```

   The RT-WiFi function lays in **Device Drivers**-> **Network device support** -> **Wireless LAN** -> **RT-WiFi support**.

   After enabling RT-WiFi support, remember to save to `.config`.

5. Execute the following commands to build the kernel including all DTBs and modules:

   ```bash
   $ make ARCH=arm64 O=$TEGRA_KERNEL_OUT -j<n>
   ```

   Where:

   `<n>` indicates the number of parallel processes to be used. A typical value is the number of CPUs in your system.

6. Replace Linux_for_Tegra/kernel/Image with a copy of:

   `$TEGRA_KERNEL_OUT/arch/arm64/boot/Image`

7. Replace the contents of Linux_for_Tegra/kernel/dtb/ with the contents of:

   `$TEGRA_KERNEL_OUT/arch/arm64/boot/dts/`

8. Execute the following commands to install the kernel modules:

   ```bash
   $ sudo make ARCH=arm64 O=$TEGRA_KERNEL_OUT modules_install INSTALL_MOD_PATH=<top>/Linux_for_Tegra/rootfs/
   ```

## Flashing the Boot Loader and Kernel

This section describes the steps that must be taken to boot the target board by flashing the kernel and boot loader (code-name Jetson TK1 platform) and provides usage information for the `flash.sh` helper script. 

### Flash Procedure
#### Prerequisites
The following directories must be present:
* /bootloader: boot loader plus flashing tools (NvFlash, CFG, BCTs, etc.)
* /kernel: a kernel zImage/vmlinux.uimg, DTB files, and kernel modules
* /rootfs: the root file system that you download (This directory starts empty and you populate it with the sample file system.) 
* /nv_tegra: NVIDIA® Tegra® user space binaries and sample applications 

You must also have the USB cable connected to the recovery port prior to running the commands listed in the procedure.

#### To flash the boot loader and kernel
1. Put the target board into reset/recovery mode. For detailed procedures, follow the instructions in [Quick Start Guide](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3231/index.html#page/Tegra Linux Driver Package Development Guide/quick_start.html).
2. Run the flash.sh script that is in the top level directory of this release. The script must be supplied with the target board (jetson-nano-devkit) for the root file system: 

   ```bash
   $ sudo ./flash.sh <platform> <rootdev>
   ```
   * If the root file system will be on a USB disk, execute the script as follows: 
      ```bash
      $ sudo ./flash.sh <platform> sda1 
      ```

   * If the root file system will be on an SD card, execute the script as follows:
      ```bash
      $ sudo ./flash.sh <platform> mmcblk1p1
      ```
   
   * If the root file system will be on the internal eMMC, execute the script as follows:
      ```bash
      $ sudo ./flash.sh <platform> mmcblk0p1 
      ```

   Where `<platform>` is jetson-nano-devkit.

 