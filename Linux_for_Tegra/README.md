# RT-WiFi kernel Installation Guide on NVIDIA Jetson Nano

**Instead of using the files from this repository, please use the files downloaded from NVIDIA.** 

You can find them in Jetson Download Center.

Download the following sources from NVIDIA:
1. L4T Jetson Driver Package
2. L4T Sample Root File System
3. L4T Sources

Extract 1 and 3 after downloading them.
---
Note: This guide only works on compiling and install on a Jetson device, not for cross compiling.

Before you get hands on compiling and install modules, make sure that you have already read the documents provided by NVIDIA.



[Quick Start Guide](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3231/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/quick_start.html)

[Developer Guide](mercury.pr.erau.edu/~siewerts/cs415/documents/Jetson/Tegra_Linux_Driver_Package_Developers_Guide.pdf) (**Just for reference! Outdated!**)

[Kernel Customization](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/kernel_custom.html#)

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

## How to make the customed kernel works?
1. Extract *L4T Sample Root File System* into `Linux_for_Tegra/rootfs/`:

```bash
sudo tar xpf <your_dir>/Tegra_Linux_Sample-Root-Filesystem_R32.5.1_aarch64.tbz2 
```

2. Archive the installed kernel modules using the following command:

```bash
cd <modules_install_path>
sudo tar --owner root --group root -cjf kernel_supplements.tbz2 lib/modules
```

3. Move the archive to `Linux_for_Tegra/kernel/`:

```bash
sudo mv kernel_supplements.tbz2  ../kernel/
```

4. Create Jetson Nano image:

```bash
cd ..
sudo ./apply_binaries.sh
cd tools
sudo ./jetson-disk-image-creator.sh -o jetson_nano.img -b jetson-nano -r 300
```

   `-r` is the revision number of the Jetson Nano module to be used:

   100 for revision A01

   200 for revision A02

   300 for revision B00 or B01

5. Using *balenaEtcher* or other software to flash the create image file into your SD card.


 