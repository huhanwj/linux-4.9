# linux-5.4-RT-WIFI
This project aims to make RT-WiFi work on newer linux kernel (i.e. 5.4 in our project).

## Notice for downloading this repository

Do NOT try to clone or download the ZIP file and unzip it under Windows Environment (Chinese Lang.), it will raise an directory error (unclear about the detailed reason but I guess is related to encoding schemes as I am using a Chinese lang version of Windows).

All of our testing is done on x86 platforms, we don't know whether this is compatible with ARM chips.

Also Github currently does not accept account passwords to authenticate Git operations, you may use SSH clone instead of HTTPS clone.

## Installation
**First, remember to download or clone the whole repository.**

### System Requirements
Ubuntu 18.04 LTS (we only test on Ubuntu machines)

### Install required packages
Some packages are needed (may still have missing packages when compiling, add the missing packages yourself depending on what error you encounter)

`sudo apt-get install build-essential bison bc flex libncurses5-dev libncursesw5-dev libssl-dev libelf-dev`

Then, run the following command in this folder

`make menuconfig`

Make sure that *RT-WiFi Support* in `Device Drivers > Network device support > Wireless LAN` is enabled (enabling by space key). Save first and then exit.

Next compile the kernel modules

```bash
make -j<n>
make modules
sudo make modules_install
sudo make install
```

Note here that the `-j<n>` is used for accelerating the compiling process. The number n is select according to the number of threads that your CPU have. If your CPU has hyperthreading technology (HT), the number should be 2 times your CPU cores (e.g. most Intel Core i7/i9 CPUs, Intel Xeon CPUs and AMD Ryzen/ThreadRipper CPUs). For CPUs not supporting such technology, which normally depends on the settings of chip manufacturer (e.g. i7-9700K does not support HT but i7-8700K does), just put in the number of your CPU cores (e.g. the CPUs used for my testing, which are Intel Core i5-4590T and Intel Core i5-9600K).

Finally, reboot the system to make it work.

## Notice for building

Do NOT put the folder under some folder with space in its name like Untitled folder, it will result in a make error during the `sudo make modules_install`.

If you find that uname -r does not give 5.4.0-RT-WIFI, then you can modify `/etc/default/grub` and deactivate `GRUB_HIDDEN_TIMEOUT=0` by commenting it. Then run `sudo update-grub` to update the grub and reboot the system. You can see the grub menu and select advanced options for ubuntu and select our customized kernel.

If you encounter the error stating

`make[1]: *** No rule to make target 'debian/canonical-certs.pem', needed by 'certs/x509_certificate_list'.  Stop.`,

you need to modify `.config` file.

In your kernel configuration file you will find this line:

`CONFIG_SYSTEM_TRUSTED_KEYS="debian/canonical-certs.pem"`

Change it to this:

`CONFIG_SYSTEM_TRUSTED_KEYS=""`

## Sources
*Source code of RT-WiFi Project*: https://www.cs.utexas.edu/users/cps/rt-wifi/RT-WiFi-v0.1.tgz

*Manual for RT-WiFi Project*: https://www.cs.utexas.edu/users/cps/rt-wifi/RT-WiFi%20user%20guide%20v0.1.pdf
