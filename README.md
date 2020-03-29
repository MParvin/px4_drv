# px4_drv - Unofficial Linux driver for PLEX PX-W3U4/Q3U4/W3PE4/Q3PE4 ISDB-T/S receivers

An unofficial Linux driver for PLEX PX-W3U4 / Q3U4 / W3PE4 / Q3PE4.
It is ** different ** from the official Linux driver distributed on PLEX's [Website] (http://plex-net.co.jp).

Currently under development, operation may not be stable depending on the environment.
Please note.

## corresponding device

- PLEX

	- PX-W3U4
	- PX-Q3U4
	- PX-W3PE4
	- PX-Q3PE4

- e-Better

	- DTV02-1T1S-U (experimental)

## Install

Before using this driver, you need to extract and install the firmware from the official driver.

### 1. Extract and install firmware

unzip, gcc, make must be installed.

	$ cd fwtool
	$ make
	$ wget http://plex-net.co.jp/plex/pxw3u4/pxw3u4_BDA_ver1x64.zip -O pxw3u4_BDA_ver1x64.zip
	$ unzip -oj pxw3u4_BDA_ver1x64.zip pxw3u4_BDA_ver1x64/PXW3U4.sys
	$ ./fwtool PXW3U4.sys it930x-firmware.bin
	$ sudo mkdir -p /lib/firmware
	$ sudo cp it930x-firmware.bin /lib/firmware/
	$ cd ../

### 2. Driver installation

Some Linux distributions may require a separate udev installation.

#### When not using DKMS

gcc, make and kernel source / header must be installed.

	$ cd driver
	$ make
	$ sudo make install
	$ cd ../

#### When using DKMS

gcc, make, kernel source / header, dkms must be installed.

	$ sudo cp -a ./ /usr/src/px4_drv-0.2.1
	$ sudo dkms add px4_drv/0.2.1
	$ sudo dkms install px4_drv/0.2.1

### 3. Verification

#### 3.1 Confirm kernel module loading

Execute the following command, and if the line starting with `px4_drv` is displayed, the kernel module has been successfully loaded.

	$ lsmod | grep -e ^px4_drv
	px4_drv                81920  0

If nothing is displayed, execute the following command and then execute the above command again to confirm.

	$ modprobe px4_drv

If the kernel module still does not load properly, start over from the installation.

#### 3.2 Checking device files

If the installation is successful and a device is connected with the kernel module loaded, a device file named `px4video *` will be created under `/ dev /`.
You can check with the following command.

	$ ls /dev/px4video*
	/dev/px4video0  /dev/px4video1  /dev/px4video2  /dev/px4video3

The tuner is assigned two S and T alternately from `px4video0`, such as ISDB-S, ISDB-S, ISDB-T, ISDB-T.

## Uninstall

### 1. Uninstall Driver

#### When installed without using DKMS

	$ cd driver
	$ sudo make uninstall
	$ cd ../

#### When installed using DKMS

	$ sudo dkms remove px4_drv/0.2.1 --all
	$ sudo rm -rf /usr/src/px4_drv-0.2.1

### 2. Uninstalling firmware

	$ sudo rm /lib/firmware/it930x-firmware.bin

## Receiving method

TS data can be received by using software that supports the chardev driver for PT series, such as recpt1 and [BonDriverProxy_Linux] (https://github.com/u-n-k-n-o-w-n/BonDriverProxy_Linux).
recpt1 does not need to use the one distributed by PLEX.

## LNB power output

It supports only no output and 15V output. By default, LNB power is not output.
To output the LNB power, add `--lnb 15` to the parameter when executing recpt1.

## Remarks

### About the built-in card reader and remote control

This driver does not support the operation of the card reader or remote control built into various compatible devices.
There are no plans to respond in the future. Please note.

### About e-Better DTV02-1T1S-U

Many problems have been reported in various places of e-Better DTV02-1T1S-U, which may not operate properly depending on the individual. Therefore, this driver is an "experimental response".
Please note that the above problems may not be completely resolved by this unofficial driver.

## Technical information

### Device configuration

The PX-W3PE4 / Q3PE4 receives power from the PCIe slot and exchanges data via USB.
The PX-Q3U4 / Q3PE4 has a structure in which two PX-W3U4 / W3PE4 equivalent devices hang down via a USB hub.

- PX-W3U4/W3PE4

	- USB Bridge: ITE IT9305E
	- ISDB-T/S Demodulator: Toshiba TC90522XBG
	- Terrestrial Tuner: RafaelMicro R850 (x2)
	- Satellite Tuner: RafaelMicro RT710 (x2)

- PX-Q3U4/Q3PE4

	- USB Bridge: ITE IT9305E (x2)
	- ISDB-T/S Demodulator: Toshiba TC90522XBG (x2)
	- Terrestrial Tuner: RafaelMicro R850 (x4)
	- Satellite Tuner: RafaelMicro RT710 (x4)

DTV02-1T1S-U shares the TS serial output of ISDB-T with ISDB-S. Therefore, only one channel can be received at a time.

- DTV02-1T1S-U

	- USB Bridge: ITE IT9303FN
	- ISDB-T/S Demodulator: Toshiba TC90532XBG
	- Terrestrial Tuner: RafaelMicro R850
	- Satellite Tuner: RafaelMicro RT710

### Configure TS Aggregation

The mode that rewrites sync_byte on the device side and distributes the TS data of each tuner on the host side based on the value is used.
