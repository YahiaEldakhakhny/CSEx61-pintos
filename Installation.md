# Installation Guidelines

This installation guidelines are tested on:

1. Qemu Simulator / Ubuntu 18.04 LTS

To be added & tested in the future:

1. Qemu Simulator / Ubuntu 20.04 LTS
2. Bochs Simulator / Ubuntu 18.04 LTS
3. Bochs Simulator / Ubutnu 18.04 LTS

## Required Dependencies

------

Check whether your machine is 32-bit or 64 bit:

* If the output is i686 or i386, then the machine is 32-bit.
* If it is x86 64, then it is 64-bit.

```shell
uname -m
```

```shell
sudo apt-get update
```

GCC. Version 4.0 or later is preferred. Version 3.3 or later should work:

* If the host machine has an 80x86 processor, then GCC should be available as gcc.
* Otherwise, an 80x86 cross-compiler should be available as i386-elf-gcc.

```shell
sudo apt-get upgrade gcc
```

GNU binutils. Pintos uses addr2line, ar, ld, objcopy, and ranlib. If the host machine is not an 80x86, versions targeting 80x86 should be available with an i386-elf- prefix.

```shell
sudo apt-get install binutils
```

```shell
sudo apt-get install pkg-config
sudo apt-get install zlib1g-dev
sudo apt-get install libglib2.0-dev
sudo apt-get install libfdt-dev
sudo apt-get install libpixman-1-dev
sudo apt-get install gcc libc6-dev
sudo apt-get install autoconf
sudo apt-get install libtool
sudo apt-get install libsdl1.2-dev
sudo apt-get install g++
sudo apt-get install libx11-dev
sudo apt-get install libxrandr-dev
sudo apt-get install libxi-dev
sudo apt-get install libnfs-dev libiscsi-dev
sudo apt-get install libc6-dbg gdb valgrind
sudo apt-get install ninja-build
```

Perl. Version 5.8.0 or later is preferred. Version 5.6.1 or later should work.

```shell
sudo apt-get upgrade perl
```

GDB. If the host machine is not an 80x86, a version of GDB targeting 80x86 should be available as i386-elf-gdb.

```shell
sudo apt-get install libc6-dbg gdb
```

GNU make, version 3.80 or later.

```shell
sudo apt-get install make
```

Git.

```shell
sudo apt-get install git
```

## Qemu Installation

------

Download Qemu project

```shell
mkdir $HOME/apps
cd $HOME/apps
git clone git://git.qemu-project.org/qemu.git
cd $HOME/apps/qemu
```

Prepare a native debug build.

```shell
mkdir -p bin/debug/native
cd bin/debug/native
```

Cinfugure your installation:

* For 64-bit machines

```shell
../../../configure --target-list=x86_64-softmmu --enable-debug
```

* For 32-bit machines

```shell
../../../configure --target-list=i386-softmmu --enable-debug 
```

If you get this error "pixman >= 0:21:8 not present.", execute the following command **then repeat the configuration step**.

```shell
git submodule update --init pixman
```

If you get this error "DTC (libfdt) version >= 1:4:0 not present.", execute the following command **then repeat the configuration step**.

```shell
git submodule update --init dtc
```

Build Qemu

```shell
make
```

Make a link of the built Qemu binary inside the /bin to be able to use it from scripts.

* For 64-bit machines

```shell
sudo ln -s $HOME/apps/qemu/bin/debug/native/x86_64-softmmu/qemu-system-x86_64 /bin/qemu
```

* For 32-bit machines

```shell
sudo ln -s $HOME/apps/qemu/bin/debug/native/i386-softmmu/qemu-system-i386 /bin/qemu
```

Check that Qemu installed correctly, the following command should print the path of Qemu /bin/qemu

```shell
which qemu
```

## PintOS Installation

------

**Any refer to $PINTOSHOME should be replaced with pintos absolute path, for example $PINTOSHOME -> /home/vm/pintos/ if username is vm.**

Assuming, that pintos is in the ~/ directory, Add PINTOSHOME to path variable.

```shell
echo "export PATH=~/pintos/src/utils:$PATH" >> ~/.bashrc
```

Reload environment variables.

```shell
source ~/.bashrc
```

GDBMACROS

Edit $PINTOSHOME/src/utils/pintos-gdb to change the definition of GDBMACROS to
point to where you installed gdb-macros. Test the installation by running the following command without any arguments. If it does not complain about missing gdb-macros, it is installed correctly.

```
Change this line GDBMACROS=/usr/class/cs140/pintos/pintos/src/misc/gdb-macros to GDBMACROS=$PINTOSHOME/src/misc/gdb-macros.
```

Example: GDBMACROS=/home/ubuntu/pintos/src/misc/gdb-macros
Note: do not forget to use the absolute path in place of $PINTOSHOME.

```shell
pintos-gdb 
```

Compile the Utilities, and ignore any warnings.

If you're using Ubuntu 20, you might have the following error: "stropts.h No such file or header", ignore squish-pty (remove its entities from the makefile) and delete the header from squish-unix and it should work fine, then run the make command again in pintos utils directory, only bochs depends on squish-pty so if you are using qemu this should not matter to you.

```
cd $PINTOSHOME/src/utils
make
```

Set Qemu as Your Simulator.

```
$PINTOSHOME/src/threads/Make.vars and change the line SIMULATOR = --bochs to SIMULATOR = --qemu
```

Compile Pintos Kernel, as a result a new "build" directory will be created as a sub-directory of: $PINTOSHOME/src/threads/.

```shell
cd $PINTOSHOME/src/threads/
make
```

Edit util Files

```
$PINTOSHOME/src/utils/pintos 
line 103: from $sim = "bochs" if !defined $sim; to $sim = "qemu" if !defined $sim;.
line 621: from my (@cmd) = ('qemu-system-i386'); to my (@cmd) = ('qemu');
line 257: from kernel.bin to the absolute path pointing to it: $PINTOSHOME/src/threads/build/kernel.bin
```

```
$PINTOSHOME/src/utils/Pintos.pm 
line 362: from loader.bin to the absolute path pointing to it: $PINTOSHOME/src/threads/build/loader.bin
```

Change pintos script permissions.

```shell
cd $PINTOSHOME/src/utils
chmod 777 pintos
```

Run Pintos & Test your installation.

```shell
pintos run alarm-multiple
cd $PINTOSHOME/src/threads
make grade
```

## Common Problems

------

1. `pintos.pm` and `pintos`, don't use variables. Use absolute paths directly. so "my $name = find_file ('$PINTOSHOME/src/threads/build/kernel.bin');" is wrong, instead you should write the absolute path for-exampe "my $name = find_file ('/usr/username/pintos/src/threads/build/kernel.bin');".

2. If Qemu was not be installed correctly, try removing "qemu" from /bin and re running the command "sudo ln -s $HOME/apps/qemu/x86_64-softmmu/qemu-system-x86_64 /bin/qemu", if this doesn't work, remove "qemu" from /bin then reinstall qemu again.

3. If you follow the provided offical pintosIntro pdf installation guide, you might copy the command directly from the document and the \_ would get discarded. Make sure you've written has the \_ in x86_64.