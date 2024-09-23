I have a Microsoft Ergonomic Keyboard. The device ID when using lsusb is 045e:082c

The keyboard is detected in the system. It works on Windows. It does not work on MacOS or Linux. I don't understand why.

I was trying to investigate and figure things out, but I saw no clear issues.

It was suggested that I configure the kernel with options for Microsoft keyboards.

All in all, I changed `CONFIG_HID_MICROSOFT=m` to `CONFIG_HID_MICROSOFT=y`.
I then had to recompile the kernel.

I stumbled through this by installing many packages to allow the kernel Makefiles to work.
I then did `nano /boot/config-$(uname -r)` to edit the file and change the value.
The "m" indicated that the Microsoft HID module is built as a module, which means it can be loaded dynamically.
I tried to load the module directly with `sudo modprobe hid-microsoft` with no luck.

I'm going to add the bash_history and try to break down everything that I did. I was very confused with regard to what I was doing, so it might be good to interpret line-by-line whaet I did.

I'll come back later to do the line by line analysis

```
mkdir keyboard_driver
cd keyboard_driver/
nano ergonomic_keyboard.c
make -C /lib/modules/$(uname -r)/build M=$(pwd) modules
sudo update-alternatives --config gcc
sudo apt update
sudo apt install build-essential software-properties-common manpages-dev
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install gcc-12 g++-12
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 1
sudo update-alternatives --set gcc /usr/bin/gcc-12
gcc --version
make -C /lib/modules/$(uname -r)/build M=$(pwd) modules
sudo update-alternatives --config gcc
make CC=x86_64-linux-gnu-gcc-12 HOSTCC=x86_64-linux-gnu-gcc-12
make -C /lib/modules/$(uname -r)/build M=$(pwd) modules
ls
sudo apt-get update
sudo apt-get install linux-headers-$(uname -r)
sudo gcc -o ergonomic_keyboard.o -c ergonomic_keyboard.c
ls /usr/src/linux-headers-$(uname -r)/include/linux/
gcc -I/usr/src/linux-headers-$(uname -r)/include -o ergonomic_keyboard.o -c ergonomic_keyboard.c
gcc -I/usr/src/linux-headers-$(uname -r)/include     -I/usr/src/linux-headers-$(uname -r)/arch/x86/include/generated     -o ergonomic_keyboard.o -c ergonomic_keyboard.c
gcc -D__NO_WARN_REDEFINES -I/usr/src/linux-headers-$(uname -r) -o ergonomic_keyboard.o -c ergonomic_keyboard.c
gcc     -I/usr/include/linux     -I/usr/src/linux-headers-$(uname -r)     -o ergonomic_keyboard.o -c ergonomic_keyboard.c
gcc     -I/usr/include/linux     -I/usr/src/linux-headers-$(uname -r)     -o ergonomic_keyboard.o -c ergonomic_keyboard.c
clear
fastfetch
sudo modprobe hid
sudo modprobe hid-generic
sudo modprobe hid-usb
dpkg -S /lib/modules/$(uname -r)/kernel/drivers/hid/hid-usb.ko
ls /lib/modules/$(uname -r)/kernel/drivers/hid/
ls /lib/modules/$(uname -r)/kernel/drivers/hid/usbhid
tree /lib/modules/$(uname -r)/kernel/drivers/hid/usbhid
sudo apt install tree
tree /lib/modules/$(uname -r)/kernel/drivers/hid/usbhid
sudo modprobe usbkbd
sudo modprobe hid
dmesg | grep -i usb
dmesg | grep -i hid
sudo dmesg | grep -i usb
sudo dmesg | grep -i hid
sudo modprobe hid-microsoft
cat /boot/config-$(uname -r) | grep -i microsoft
zcat /proc/config.gz | grep -i microsoft
nano /boot/config-$(uname -r)
sudo nano /boot/config-$(uname -r)
sudo modprobe hid-microsoft
lsmod | grep hid_microsoft
sudo modprobe usbhid
sudo modprobe usbkbd
sudo modprobe hid-microsoft quirks=quirk_ignore_invalid_id
lsusb
sudo sysctl -a | grep -E "MICROSOFT"
sudo nano /etc/default/grub
lsmod | grep microsoft
sudo nano /boot/config-$(uname -r)
sudo make -j $(nproc)
dpkg -l | grep linux-source
sudo apt-get install linux-source
find / -name "Makefile" -type f 2>/dev/null | xargs grep -i "VERSION"
cd /boot
ls
uname -r
cd /usr/src
ls
cd linux-headers-6.9.3-76060903-generic/
ls
sudo make -j $(nproc)
make menuconfig
sudo make menuconfig
pwd
sudo apt update
sudo apt install build-essential flex bison libssl-dev libelf-dev
sudo make -j $(nproc)
sudo nano /boot/config-$(uname -r)
sudo cp /boot/config-6.9.3-76060903-generic /boot/.config
cd /usr/src/linux-headers-6.9.3-76060903-generic
sudo make clean
sudo make M=drivers/hid/hid-microsoft
sudo cp /boot/config-6.9.3-76060903-generic .config
ls
ls -la
rm .config
cd .config
ls
ls -l
cd /usr/src/linux-headers-6.9.3-76060903-generic
sudo cp /boot/config-6.9.3-76060903-generic .config
sudo nano .config
sudo make M=drivers/hid/hid-microsoft
find /usr/src/linux-headers-6.9.3-76060903-generic -name "*.c" | xargs grep -H "CONFIG_HID_MICROSOFT"
ls drivers/hid
cd drivers/hid/hid-microsoft/
ls
sudo apt-get install linux-headers-generic
touch drivers/hid/hid-microsoft/Makefile
sudo chmod 644 drivers/hid/hid-microsoft/Makefile
cd ../../..
ls
sudo chmod 644 drivers/hid/hid-microsoft/Makefile
touch drivers/hid/hid-microsoft/Makefile
sudo touch drivers/hid/hid-microsoft/Makefile
sudo chmod 644 drivers/hid/hid-microsoft/Makefile
sudo make M=drivers/hid/hid-microsoft
```

Articles that helped: 
[Gentoo Forums](https://forums.gentoo.org/viewtopic-p-7354130.html)
[Hardware Linux, which I don't understand](https://linux-hardware.org/?id=usb:045e-082c)
