# How to create a new ramdisk image

## On Linux:
* First, we need to install all necessary tools that will be used to create and manipulate our ramdisk image:

   `sudo apt-get install hfsplus hfsprogs`

* Next, we will use the `dd` utility to create a raw 32MB ramdisk image and format that image as `hfs+`:

   ```
   dd if=/dev/zero of=./ramdisk_new.dmg bs=1M count=32
   mkfs.hfsplus ./ramdisk_new.dmg
   ```

* Next, we can mount the image using the following commands:

   ```
   mkdir ./tmp
   sudo mount -t hfsplus -o loop ./ramdisk_new.dmg ./tmp
   ```

* Next, we can populate our new ramdisk image. For this step we will copy the contents of our preexisting ramdisk to our new image:

   ```
   git clone https://github.com/darwin-on-arm/ramdisk.git
   mkdir ./tmp2
   sudo mount -t hfsplus -o loop ./ramdisk/ramdisk.dmg ./tmp2
   sudo cp -ap ./tmp2/* ./tmp/
   ```

* Finally, we can unmount our images and create a new `Ramdisk.img3` file that can be used by GenericBooter:
   ```
   sudo umount tmp
   sudo umount tmp2
   image3maker -t rdsk -f ramdisk_new.dmg -o Ramdisk.img3
   ```

  To use your newly created `Ramdisk.img3` file, place it under the `images` directory within GenericBooter and rebuild it.



## On OSX:

* First, we will use the `hdiutil` utility to create a raw 32MB ramdisk image formatted as `hfs+`:
   ```
   hdiutil create -size 32m -type UDIF -fs JHFS+ -volname "Ramdisk" ./ramdisk_new.dmg
   ```

* Next, we can mount the image using the `open` command:

   ```
   sudo open ./ramdisk_new.dmg
   ```
   A new empty finder window should pop up. This is the root directory of your new ramdisk. 

* Next, we can populate our new ramdisk image. For this step we will copy the contents of our preexisting ramdisk to our new image:

   First, we need to obtain our preexisting ramdisk if we don't already have it:

   `git clone https://github.com/darwin-on-arm/ramdisk.git`

   Then we need to mount our preexisting ramdisk using the `open` command:

   `sudo open ./ramdisk/ramdisk.dmg`

   A new finder window should pop up displaying the filesystem of our preexisting ramdisk.

   Finally, we can copy over the contents from our preexisting ramdisk to our new ramdisk. To accomplish this, click inside of the preexisting ramdisk's finder folder and press command-a, then while holding down the mouse drag the selected folders over to the new window.

* Finally, we can umount our images using the `hdiutil` utility and create a new `Ramdisk.img3` file that can be used by GenericBooter:

   ```
   sudo hdiutil detach ramdisk/ramdisk.dmg
   sudo hdiutil detach ramdisk_new.dmg
   image3maker -t rdsk -f ramdisk_new.dmg -o Ramdisk.img3
   ```

  To use your newly created `Ramdisk.img3` file, place it under the `images` directory within GenericBooter and rebuild it.
