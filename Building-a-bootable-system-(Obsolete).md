IMPORTANT NOTICE: This page contains outdated information and is only available for legacy purposes!
Please visit the Building Darwin page for current information.

### Getting started with `xnu` on Mac OS X.

Make sure you at least have the following installed:

* Xcode 4.x or later.
* `dtrace-96` (You'll need `ctfmerge`, `ctfdump` and `ctfconvert`)
* `kextsymboltool` and `setsegname` (Should usually come from Xcode command line tools.)

***

### Getting started with `xnu` on Linux/BSD.

Make sure you have the following installed

* `xnu-deps-linux` (Available [here](http://github.com/darwin-on-arm/xnu-deps-linux).)
* All dependencies for `xnu-deps-linux`.

Alternatively, you may also use a chroot, available [here](https://github.com/stqism/xnu-chroot-x86_64) for 64-bit Linux systems.

***

### Compiling the kernel

First, you have to check out the source code and place it into a working directory. The `xnu-build` script may
be used to perform full-builds of the kernel. (includes all supported targets/definitions as specified in the `machine_configuration` file)

Compiling the kernel is simple enough:

<pre>
[rms@fujimoto /home/rms/xnu]$ make TARGET_CONFIGS="debug arm armpba8"
</pre>

(And if you're so opposed to figuring out how to type a `git clone` command...)

<pre>
[rms@fujimoto /home/rms]$ git clone git://github.com/darwin-on-arm/xnu.git
Cloning into 'xnu'...
remote: Counting objects: 7917, done.
...
[rms@fujimoto /home/rms]$ cd xnu
</pre>

A resulting kernel will be located in your object root folder. This will usually be the `BUILD/obj` directory, however, if you are using the `xnu-build` script, your object root folder will be located in `/private/var/tmp/xnu/Objects`.

At this point, you can either proceed to prelink the kernel with kernel extensions, or you can just turn the kernel image into an image3 file. (The file magic is `krnl` for the kernel.)

***

### Generating Image3 files

You can either use [xpwn](http://github.com/planetbeing/xpwn) or [image3maker](http://github.com/darwin-on-arm/image3maker) to generate Image3 files. Usage should be simple enough to figure out on your own.

***

### Compiling the Bootloader

Currently, GenericBooter is used to start up Darwin/ARM systems. This simple boot loader converts Linux boot ATAGs into Darwin boot-args/devicetree boot structures.

A version for the BeagleBone (Black) is available at http://github.com/furkanmustafa/GenericBooter.

The other version, hosted on the Darwin-on-ARM repository only supports ARM RealView and OMAP3530. It is available [here](http://github.com/darwin-on-arm/GenericBooter).

You will need the `arm-none-eabi-` toolchain. Currently, the Linaro toolchain is preferred. Additionally, `mkimage` from u-boot is also required.

Place your ramdisk as `rdsk.img3` inside the GenericBooter folder with magic `rdsk` and the kernel (uncompressed kernelcache if used) as `mach.img3` (magic `krnl`).

Then, simply use `make` to build the resulting bootable image.

***

### Booting the kernel

The kernel can be started up from u-boot as either an XIP image (if the text base begins at `LOAD_ADDRESS+0x40`) or a relocatable image.

It can be loaded from any device. Booting is done like so:

<pre>
u-boot# fatload mmc 0 0x84000000 /SampleBooter.elf.uImage
u-boot# setenv bootargs rd=md0 debug=0x16e serial=3 -v -s
u-boot# bootm 0x84000000
</pre>

Hopefully, if everything works correctly, you will be able to boot into a simple Darwin/ARM system.

Have fun~.