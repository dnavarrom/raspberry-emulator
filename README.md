# raspberry-emulator
QEMU raspberry emulator setup

Tested on Linux Mint 18 64-Bits

## Step 1: Setup QEMU

` sudo apt-get install qemu `

verify that qemu-arm is present (qemu- and double TAB)


## Step 2: Download Raspberry PI Lite Image

Download the LITE image "Minimal image based on Debian Jessie"

https://www.raspberrypi.org/downloads/raspbian/

rename it to rpi.img

## Step 3: Download QEMU Raspberry PI Kernel 

https://github.com/dhruvvyas90/qemu-rpi-kernel

you should have the following folder structure

```
.
├── 
├── qemu-rpi-kernel
│   ├── kernel-qemu-3.10.25-wheezy
│   ├── kernel-qemu-4.1.13-jessie
│   ├── kernel-qemu-4.1.7-jessie
│   ├── kernel-qemu-4.4.12-jessie
│   ├── kernel-qemu-4.4.13-jessie
│   ├── kernel-qemu-4.4.21-jessie
│   ├── kernel-qemu-4.4.26-jessie
│   ├── kernel-qemu-4.4.34-jessie
│   ├── README.md
│   └── tools
│       ├── build-kernel-qemu
│       ├── config_file_3.10.25
│       ├── config_ip_tables
│       ├── linux-arm.patch
│       ├── qemu_choose_vm.sh
│       └── README.md
└── rpi.img
```


## Step 4: Configure startup script

Create a new file and name it config

```
#!/bin/bash
# Starts raspberry pi image in configuration mode

qemu-system-arm -kernel ./qemu-rpi-kernel/kernel-qemu-4.4.13-jessie \
-cpu arm1176 \
-m 256 \
-M versatilepb \
-no-reboot \
-serial stdio \
-append "root=/dev/sda2 panic=1 rootfstype=ext4 rw init=/bin/bash" \
-hda rpi.img
```
add execution permissions
`sudo chmod +x config`

## Step 5: Run the basic machine

`./config`

Done, you should get the # inside RPI Machine, If QEMU window has captured your mouse, you can release it by pressing LEFT CTRL + LEFT ALT



## Step 6: RPI OS Config

Mount the image file 
```
fdisk -l ./rpi.img

Device     Boot Start     End Sectors  Size Id Type
rpi.img1         8192   92159   83968   41M  c W95 FAT32 (LBA)
rpi.img2        92160 2534887 2442728  1,2G 83 Linux
```

The filesystem (.img2) starts at sector 92160, which equals 512 * 92160 = 47185920 bytes. 

* Use this value in the offset parameter

```
sudo mkdir /mnt/rpi
sudo mount -v -o offset=47185920 -t ext4 rpi.img /mnt/rpi
```

* Comment out every entry in ld.so.preload file
``` 
 cd /mnt/rpi
 sudo nano ./etc/ld.so.preload 
```
* Define some rules to tell raspberry pi how to direct calls to SD-Card to the “sda” 

```
sudo nano ./etc/udev/rules.d/90-qemu.rules

KERNEL=="sda", SYMLINK+="mmcblk0"
KERNEL=="sda?", SYMLINK+="mmcblk0p%n"
KERNEL=="sda2", SYMLINK+="root"
```

* Umount volume and go back to the working folder
`cd ~`
`sudo umount /mnt/rpi`
`cd path/to/folder`

* Create startup script the config script, change the --append line removing init=/bin/bash

```
cp config start
nano ./start
```

```
#!/bin/bash
# Starts raspberry pi image in configuration mode

qemu-system-arm -kernel ./qemu-rpi-kernel/kernel-qemu-4.4.13-jessie \
-cpu arm1176 \
-m 256 \
-M versatilepb \
-no-reboot \
-serial stdio \
-append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" \
-hda rpi.img
```

* Run QEMU again, you should get booted into a fully functional emulated Raspberry Pi

```
./start
```

* Login 

Username: pi 
Password: raspberry


## If your raspibian distro always reboot

References:
* https://github.com/dhruvvyas90/qemu-rpi-kernel/issues/31
* https://github.com/dhruvvyas90/qemu-rpi-kernel/wiki/Emulating-Jessie-image-with-4.x.xx-kernel

Basically you need to modify your Raspibian image:

```
fdisk -l ./rpi.img

Device     Boot Start     End Sectors  Size Id Type
rpi.img1         8192   92159   83968   41M  c W95 FAT32 (LBA)
rpi.img2        92160 2534887 2442728  1,2G 83 Linux
```

The filesystem (.img2) starts at sector 92160, which equals 512 * 92160 = 47185920 bytes. 

```
sudo mkdir /mnt/rpi
sudo mount -v -o offset=47185920 -t ext4 rpi.img /mnt/rpi
```

` cd /mnt/rpi`
` sudo nano ./etc/ld.so.preload `

* Comment out every entry in it, Ctrl-x » Y to save and exit

` sudo nano ./etc/fstab `

* Comment out entries containing /dev/mmcblk, Ctrl-x » Y to save and exit
`cd ~`
`sudo umount /mnt/rpi`

Run config script again and get the prompt
