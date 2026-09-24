# Darwin Build Guide

###### Note: This guide is a work-in-progress and *will* contain incomplete information.
Contributions to improve this guide are welcome and appreciated.

###### This guide will cover the following steps:

   - Getting the build environment ready
   - Retrieving source code for the kernel and bootloader
   - Cross compiling the kernel
   - Assembling a bootable darwin image
   - Basic boot and debug procedures

#### Step 1: Getting the build environment ready
Before we can jump in and start building things, we must first get the build environment ready.

###### Setup on Mac OSX: (Work in progress)
1. Install Xcode and the iOS SDK.
2. Install Mac Ports. Follow the instructions on their website: https://www.macports.org/
3. Install git and image3maker. To install git and image3maker, run the following commands:

    ```
    mkdir -p ~/Projects/DarwinARM/Work; cd ~/Projects/DarwinARM/Work
    git clone https://github.com/darwin-on-arm/image3maker.git
    cd image3maker; make
    sudo install -s -m 755 image3maker /usr/bin/image3maker
    sudo port install git
    ```
    
###### Setup on Linux:
1. Install the Darwin SDK. To install the darwin SDK, run the following commands:
    ```
    sudo apt-get install git-core qemu-system-arm
    mkdir -p ~/Projects/DarwinARM/Work; cd ~/Projects/DarwinARM/Work
    git clone https://github.com/darwin-on-arm/darwin-sdk.git
    
    # If you are on Debian, Ubuntu or Mint:
    sudo apt-get install dpkg-dev devscripts debhelper clang-3.4 llvm-dev \
    uuid-dev libssl-dev libblocksruntime-dev libc6-dev-i386 \
    gcc-multilib build-essential flex tcsh bison automake \
    autogen libtool gobjc libdb-dev u-boot-tools
    cd darwin-sdk; dpkg-buildpackage -uc -B; cd ..
    sudo dpkg -i darwin-sdk_*.deb
    
    # For other distros, install all equivelent packages that are
    # listed above and run the following:
    cd darwin-sdk; sudo make install; cd ..
    ```
2. Install the linaro bare metal toolchain: http://www.linaro.org/downloads/ Unpack it and add it to your `PATH` environment variable or install the `arm-none-eabi` toolchain from apt-get if on Debian, Ubuntu or Mint.

You should now have all of the tools necessary to proceed to the next step: Retrieving source code for the kernel and bootloader.

#### Step 2: Retrieving source code for the kernel and bootloader

Run the following commands:

```
git clone https://github.com/darwin-on-arm/xnu.git
git clone https://github.com/darwin-on-arm/GenericBooter.git
```

#### Step 3: Cross compiling the kernel
Depending on the desired build target, these steps may vary. 
For now, we will assume that the RealView Platform Builder A8 (armpba8) is our target.
A full list of other supported platforms can be found in the `machine_configuration` file within the kernel's source directory. We will also assume to build a debug kernel for development purposes.

To build the the kernel, run the following commands:
```
cd xnu; make TARGET_CONFIGS="debug arm armpba8" NO_DTRACE_SYMS=YES; cd ..
```

#### Step 4: Building a bootable Darwin image
As with the previous step, this step may vary depending on the desired build target. 
For platforms that support U-Boot or are emulated via qemu, follow the steps listed below.
Steps for other platforms such as the iPhone 4 that don't require GenericBooter are comming soon.

###### On Linux:

   * You need the Apple device tree compiler.
       edit the `Makefile` and change `DESTDIR =` with a destination path. then run the commands:

   ```
   git clone https://github.com/darwin-on-arm/dtc-AppleDeviceTree.git
   cd dtc-AppleDeviceTree; make; sudo make install; cd ..
   ```

       once done add the destination path `bin` folder to your `PATH` environment variable
   * In order to build the bootloader, you need a few things:

   ```
   git clone https://github.com/darwin-on-arm/ramdisk.git
   git clone https://github.com/darwin-on-arm/DeviceTrees.git
   cd DeviceTrees; make; cd ..
   ```

   * To build the bootloader, run the following commands:

   ```
   cd GenericBooter
   make menuconfig
   # Make sure "Cortex-A8" is selected for ARM processor target
   # Make sure "ARM RealView" is selected for ARM board target
   # Make sure "iOS-compatible flattened device tree" is selected for Device tree style
   
   # Create kernel IMG3
   image3maker -t krnl -f ../xnu/BUILD/obj/*/mach_kernel -o images/Mach.img3
   # Create device tree IMG3
   image3maker -t dtre -f ../DeviceTrees/RealView.devicetree -o images/DeviceTree.img3
   # Create ramdisk IMG3
   image3maker -t rdsk -f ../ramdisk/ramdisk.dmg -o images/Ramdisk.img3
   # Cross-compile the bootloader
   make CROSS_COMPILE=arm-none-eabi-
   cd ..
   ```
   
   If you've followed these steps successfully, you should now have a bootable uImage file.

###### On Mac OSX: (Work in progress)
   *Coming soon!*

#### Step 5: Basic boot and debug procedures
###### Booting Darwin:
- If you are booting Darwin via qemu, run the following command:

   ```
    qemu-system-arm -s -S -machine realview-pb-a8 -m 512 \
    -kernel GenericBooter/uImage \
    -append "rd=md0 debug=0x16e serial=3 -v symbolicate_panics=1" \
    -serial stdio
   ```
- If you are booting Darwin from u-boot, format the first partition of your SD card, place your uImage into that partition, and run the following commands at the U-Boot prompt:

   ```
    # Assuming that you are booting Darwin on a Beaglebone/Beaglebone Black (load addr: 0x84000000):
    fatload mmc 0 0x84000000 /uImage
    setenv bootargs rd=md0 debug=0x16e serial=3 -v symbolicate_panics=1
    bootm 0x84000000
   ```
- If you are booting darwin on an A4-based device (i.e: Apple iPhone 4), run the following commands:
*Coming soon!*

If all goes well, you should be greeted with a `login:` prompt. Type root and press enter to gain access to the bash shell. If you encounter a problem, read over the guide once more to ensure that you haven't overlooked anything. If the problem still persists, open an issue report and we will see what we can do for you.

###### Debugging Darwin:
*Coming soon!*

[Back to Home](https://github.com/darwin-on-arm/wiki/wiki)