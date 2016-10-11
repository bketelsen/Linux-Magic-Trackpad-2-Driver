# Linux-Magic-Trackpad-2-Driver

The driver for the Apple Magic Trackpad 2 can be found in my cloned linux repo:
https://github.com/robotrovsky/linux

USB Device ID is registered in:
```
drivers/hid/hid-ids.txt
drivers/hid/hid-apple.c
drivers/hid/hid-core.c
```

The modified MacBookPro Touchpad driver is bcm5974:
```
drivers/input/mouse/bcm5974.c
```


See the first commit:
https://github.com/robotrovsky/linux/commit/7b50169c3a8948e67a67eb530b91117a7f5d9d5b

Future modifications will only affect bcm5974.c.

I used this driver in combination with the mtrack-driver, to make use of the multitouch feature (e.g. 2-finger-tap for right-click, etc):
https://github.com/BlueDragonX/xf86-input-mtrack

Right now the driver only works over USB. 

#Installation on Arch Linux:
You have to compile the kernel modules. 
See https://wiki.archlinux.org/index.php/Kernels/Arch_Build_System. The clean way would be to create a patch from the changes in
https://github.com/robotrovsky/linux/commit/7b50169c3a8948e67a67eb530b91117a7f5d9d5b and add that patch to the prepare routine. A quick unclean way would be as follows:

```
 $ cd ~/
 $ mkdir build
 $ cd build/
 $ ABSROOT=. abs core/linux
 $ cd core/linux
```

Edit 

> PKGBUILD

 and look for the pkgbase parameter. Change this to "linux-custom":

` pkgbase=linux-custom`

This is important otherwise you will later on overwrite your running kernel!!

Then:
```
$ updpkgsums
$ makepkg -s --skippgpcheck
```


When the makepkg has downloaded the kernel applied the patches and begins to compile just hit "Ctrl C" to stop the creation of the package and the compilation.

Now apply the changes to your source: 

` $ cd src/linux-{kernel-version}/`

Insert the hid-id in
```
drivers/hid/hid-ids.h
drivers/hid/hid-apple.c
drivers/hid/hid-core.c
```
Replace the bcm5974-driver source with the modified one 
`drivers/input/mouse/bcm5974.c`
with
https://raw.githubusercontent.com/robotrovsky/linux/master/drivers/input/mouse/bcm5974.c

Then:
```
$ make modules -j{number of cores}
$ sudo make modules_install
```

Now you have recompiled all kernel modules and installed them to` /lib/modules/{kernel-version}-custom/`

Make a backup of your original `drivers` folder and just copy the folder `/lib/modules/{kernel-version}-custom/kernel/drivers` over to your ARCH Kernel `/lib/modules/{kernel-version}-ARCH/kernel/drivers`

After a reboot your Magic Trackpad 2 should work. You can check, that it now uses the bcm5974 driver with `usb-devices`

Now you should install the mtrack driver
`xf86-input-mtrack`
is the package-name in yaourt.

Then adjust your configuration like explained here:
https://github.com/BlueDragonX/xf86-input-mtrack/blob/master/README.md


My 
```
/etc/X11/xorg.conf.d/10-mtrack.conf
``` 
is still not perfect, but you can use it as a starting point. It supports horizontal scrolling in the natural direction, but i havent configured yet the 3 finger drag.

```
Section "InputClass"
    MatchIsTouchpad "on"
    Identifier "Touchpads"
    Driver "mtrack"
    Option          "Sensitivity" "0.64"
    Option          "FingerHigh" "5"
    Option          "FingerLow" "1"
    Option          "IgnoreThumb" "true"
    Option          "IgnorePalm" "true"
    Option          "DisableOnPalm" "true"
    Option          "TapButton1" "1"
    Option          "TapButton2" "3"
    Option          "TapButton3" "2"
    Option          "TapButton4" "0"
    Option          "ClickFinger1" "1"
    Option          "ClickFinger2" "2"
    Option          "ClickFinger3" "3"
    Option          "ButtonMoveEmulate" "false"
    Option          "ButtonIntegrated" "true"
    Option          "ClickTime" "25"
    Option          "BottomEdge" "30"
    Option          "ScrollDistance" "75"
    Option          "VertScrollDelta" "-111"
    Option          "HorizScrollDelta" "-111"
    Option          "ScrollLeftButton" "7"
    Option          "ScrollRightButton" "6"

EndSection
```