# This guide is the full manual install process. A scripted/automated install is available for Fates.

see [Fates](https://github.com/okyeron/fates) instructions if you're using a Fates board.

# compile instructions for norns on raspberry pi with 4.14.y-rt

Starting from a fresh Stretch image (from etcher)

```
sudo raspi-config
	Change password: sleep
	Network > Hostname (norns)
	Interfacing > SSH (on)
	Interfacing > i2c (on)
	Interfacing > SPI (on)
	Advanced > Expand File System
	Localization > (en-US-UTF8, US-UTF8)
	Localization Options > Change Timezone
	Localization WiFi Country > Change Timezone	
```	

Finish, Reboot

Change user name to `we`

```
sudo passwd root
```
logout, login as root (best to do this with keyboard directly connected)
```
usermod -l we -d /home/we -m pi
groupmod --new-name we pi
```
Exit, login as we (Can jump back to SSH here)
```
sudo passwd -l root
```

Disable need for passwd with sudo - look for 010_pi-nopasswd or similar
```
ls /etc/sudoers.d/
ssudo nano /etc/sudoers.d/010...
```
Change `pi` to `we`

Run updates, install git, build dependencies, etc
```
cd ~
sudo apt-get update 
sudo apt-get dist-upgrade
sudo apt-get install vim git bc i2c-tools 
sudo apt-get -y install libncurses5-dev 

git clone https://github.com/okyeron/norns-linux-bits.git

git clone --depth 1 --branch rpi-4.14.y-rt https://github.com/raspberrypi/linux
cd linux
```
we want commit hash `22bb67b8e2e809d0bb6d435c1d20b409861794d2` so...
```
git checkout 22bb67b8e2e809d0bb6d435c1d20b409861794d2
```

copy files from my `drivers-staging-fbtft` folder to 

```

cp ~/norns-linux-bits/drivers-staging-fbtft/* /home/we/linux/drivers/staging/fbtft/
    
```


copy my `bcm2709_defconfig` from `arch-arm-configs` folder to 

```
cp ~/norns-linux-bits/arch-arm-configs/bcm2709_defconfig /home/we/linux/arch/arm/configs/bcm2709_defconfig

```

(repalcing the one that's there)

copy my `.config` to

```
cp ~/norns-linux-bits/.config /home/we/linux/.config
```



Then compile

```
# Build kernel for Pi 2, Pi 3, Pi 3+ and Compute Module 3

cd ~/linux

export KERNEL=kernel7
make mrproper
make bcm2709_defconfig 


make modules_prepare

# just to check that CONFIG_FB_TFT_SSD1322=m
make menuconfig
        	Device Drivers  ---> Staging Drivers ---> Support for small TFT LCD display modules  --->
        	<M>   SSD1322 driver


make -j4 zImage modules dtbs
sudo make modules_install
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$KERNEL.img

```

and reboot

Test ssd1322: (adjust if using different GPIOs)
```
sudo modprobe fbtft_device custom name=fb_ssd1322 width=128 height=64 speed=16000000 gpios=reset:15,dc:14
```
`lsmod` to check loading should show
```
fb_ssd1322             16384  0
fbtft_device           49152  0
fbtft                  45056  2 fbtft_device,fb_ssd1322
```


`con2fbmap 1 1` to show terminal console on OLED or `con2fbmap 1 0` to go back to HDMI (fb0) - NOTE OLED will not clear until you reboot or do some other screen activity


Then copy overlay .dtbo files for norns

```
sudo cp ~/norns-linux-bits/overlays/norns-buttons-encoders.dtbo /boot/overlays
sudo cp ~/norns-linux-bits/overlays/ssd1322-fates.dtbo /boot/overlays/
```

See norns install instuctions for how to enable the ssd1322 in norns-image service modules.
