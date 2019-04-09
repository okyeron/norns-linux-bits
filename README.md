# compile instructions for norns on raspberry pi with 4.14.y-rt

Starting from a fresh copy of Stretch

```
cd ~
git clone https://github.com/okyeron/norns-linux-bits.git

git clone --depth 1 --branch rpi-4.14.y-rt https://github.com/raspberrypi/linux
cd linux
```
we want commit hash `22bb67b8e2e809d0bb6d435c1d20b409861794d2` so...
```
git checkout 22bb67b8e2e809d0bb6d435c1d20b409861794d2
```

copy files from `drivers-staging-fbtft` folder to 

```
cd ~/norns-linux-bits
cp norns-linux-bits/drivers-staging-fbtft/* /home/we/linux/drivers/staging/fbtft/
    
```


copy `bcm2709_defconfig` from `arch-arm-configs` folder to 

```
cp norns-linux-bits/arch-arm-configs/bcm2709_defconfig /home/we/linux/arch/arm/configs/bcm2709_defconfig

```

(repalcing the one that's there)

copy `.config` to

```
cp norns-linux-bits/.config /home/we/linux/.config
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

make -j4 zImage modules dtbs
sudo make modules_install
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$KERNEL.img

```

and reboot

then copy overlay .dtbo files later
