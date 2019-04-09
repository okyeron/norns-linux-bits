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

then copy overlay .dtbo files later

```
sudo cp ~/norns-linux-bits/overlays/norns-buttons-encoders.dtbo /boot/overlays
sudo cp ~/norns-linux-bits/overlays/ssd1322-fates.dtbo /boot/overlays/
```
