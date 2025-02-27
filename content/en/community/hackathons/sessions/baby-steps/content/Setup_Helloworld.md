It is **heavily** recommended that for building and developing applications with Unikraft, you use a conventional folder structure.
Namely, for the examples that we will be going through in this session, our working directory will look like this:

```console
workdir                     ---> working directory containing applications
`-- helloworld              ---> application directory
    |
    |
    |-- main.c              ---> application code
    |-- Makefile            ---> makefile for specifying external library dependencies
    |-- Config.uk           ---> unikraft configuration options for the application
    |-- Makefile.uk         ---> the application's compiler flags and source files to build
    |
    |
    `-- .unikraft
        |-- libs        ---> external libraries used by the application
        `-- unikraft    ---> unikraft core kernel, containing all the internal libraries
`-- nginx
    |
    |
    |
`-- python3
    |
    |
    |
...
```

Let's begin with the simplest app there is, `helloworld`.
We first create the `workdir` folder where we will place all of our applications and clone the [`app-helloworld`](https://github.com/unikraft/app-helloworld/) repository, which contains the source files and the Makefiles necessary for bulding the `helloworld` application.

```console
$ mkdir workdir

$ cd workdir/

$ git clone https://github.com/unikraft/app-helloworld helloworld

$ cd helloworld/
[...]

$ ls -F
[...]  Makefile  Makefile.uk  README.md  main.c  [...]
```

Now, while inside the `helloworld/` directory, we create the `.unikraft/` folder, which will store the Unikraft core kernel from this [repository](https://github.com/unikraft/unikraft), along with the `libs/` directory which should store all of the external libraries the applications depends on, cloned from the `lib-` repos of the [Unikraft organization](https://github.com/unikraft/).
Since the `helloworld` application is fairly simple, it does not use any external library, so we can leave the `libs/` directory empty.

```console
$ mkdir .unikraft

$ cd .unikraft/

$ git clone https://github.com/unikraft/unikraft unikraft

$ mkdir libs
```

### Configuration

Now that we have our setup ready, we can head back to the `helloworld/` directory and start configuring our application!

```console
$ cd ../

$ make menuconfig
```

This will open a menu-driven user interface in which you can choose the target architecture, virtualization platform, and configuration options for both internal and external libraries of the resulting unikernel image.

We are met with the following configuration menu. Let's pick the x86 architecture:

![arch selection menu](/community/hackathons/sessions/baby-steps/images/menuconfig_select_arch.png)

![arch selection menu2](/community/hackathons/sessions/baby-steps/images/menuconfig_select_arch2.png)

![arch selection menu3](/community/hackathons/sessions/baby-steps/images/menuconfig_select_arch3.png)

Now, press `Exit` (or hit the `Esc` key twice) until you return to the initial menu.

We have now set our desired architecture, let's now proceed with the platform.
We will choose only the `KVM` platform (press the `Y` key to select it), which is supported by QEMU, the program we will use to launch our virtual machine after compilation:

![plat selection menu](/community/hackathons/sessions/baby-steps/images/menuconfig_select_plat.png)

![plat selection menu2](/community/hackathons/sessions/baby-steps/images/menuconfig_select_plat2.png)

For the moment, this is sufficient. Press the `Save` button and exit the configuration menu by repeatedly selecting `Exit`.

### Building
We can now build our application. We can do this by simply invoking `make`:

```console
$ make -j$(nproc)
```

This command invokes the build system by compiling the application on `$(nproc)` different threads, for shorter build times.
The compilation process includes all of the Unikraft core internal libraries selected by default for this app, along with the application source files (just `main.c` for `helloworld`).
The process should end with the following output:

```console
[...]
  LD      helloworld_qemu-x86_64.dbg
  UKBI    helloworld_qemu-x86_64.dbg.bootinfo
  SCSTRIP helloworld_qemu-x86_64
  GZ      helloworld_qemu-x86_64.gz
make[1]: Leaving directory '/home/X/workdir/helloworld/.unikraft/unikraft'
```
Notice that each line in `make`'s output consists of an operation in the build process (i.e compiling, linking, symbol striping etc.) and a corresponding generated file.

### Running
Now that the unikernel image has been sucessfully generated, we can finally run it using the QEMU Virtual Machine Monitor:

```console
$ qemu-system-x86_64 -kernel build/helloworld_qemu-x86_64 -nographic
```

The `-kernel` option is used for indicating the image to be booted by the machine, while the `-nographic` option is simply for using QEMU as a command-line program by redirecting all output to the terminal.

```console
SeaBIOS (version Arch Linux 1.16.2-1-1)


iPXE (http://ipxe.org) 00:03.0 C900 PCI2.10 PnP PMM+06FD3260+06F33260 C900



Booting from ROM..Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~5eb820bd
Hello world!
Arguments:  "build/helloworld_qemu-x86_64"
```

If everything went well, you should see the beautiful Unikraft banner and the "Hello world" message printed out.
Congratulations!

### Choosing another architecture

Let's try to build the `helloworld` app for the ARM64 architecture, as well.
All you need to do is head back into the configuration menu by running `make menuconfig`, after which you should go to the `Architecture Selection` menu where you can pick the `Armv8 compatible (64 bits)` target.
Make sure to save your changes.
When you exit the menu, you will most likely see this warning message:

```console
*** The following configuration changes were detected:
*** - CPU architecture changed
*** - CPU optimization changed
*** Execute 'make clean' before executing 'make'.
*** This is to ensure that the new setting is applied
*** to every compilation unit.
```
This is a very good recommandation and you'll use it a lot when configuring and building applications - so make sure to run

```console
$ make properclean
```

...followed by

```console
$ make -j$(nproc)
```

...to finally build an ARM64 unikernel.
Since we are targeting a different architecture, we have to use QEMU to emulate an ARM64 CPU:

```console
$ qemu-system-aarch64 -kernel build/helloworld_qemu-arm64 -nographic -machine virt -cpu cortex-a57
```

The `-kernel` and `-nographic` are the same as in the x86 case.
The `-machine` option is used to specify which `ARM` board we want to emulate. We can choose among Raspberry PI, Siemens, NXP boards and so on, but we are instead using the generic platform `virt`, which does not correspond to real hardware and is the board [recommended by QEMU](https://www.qemu.org/docs/master/system/arm/virt.html) to run guests.
Lastly, the `-cpu` option is used for specifying the CPU whose ISA we want to emulate.
Some CPUs differ, for example, with regards to the [extensions](https://sourceware.org/binutils/docs/as/AArch64-Extensions.html) they implement.
The `cortex-a57` is sufficient for our examples.

If everything went well, you should be greeted by the same Unikraft banner:

```console
Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~5eb820bd
Hello world!
Arguments:  "helloworld"
```
### Enabling kernel logs

We will now try to further configure our `helloworld` application.
For example, we can try to enable debug messages logged by the kernel to gain a better undersanding of the whole booting process.
You can target either ARM64, or x86 - it's your choice.
We can do this through the same `menuconfig` interface presented earlier, but first, it is recommended that you clean all of the application's build files in order to make sure that the changes are applied correctly.

```console
$ make properclean

make[1]: Entering directory '/home/X/workdir/helloworld/.unikraft/unikraft'
  RM      build/
make[1]: Leaving directory '/home/X/workdir/helloworld/.unikraft/unikraft'

$ make menuconfig
```

To enable debug messages, we need to configure Unikraft's internal library which handles logging information, `ukdebug`. To do this, we can proceed to the `Library Configuration` menu:

![library selection menu](/community/hackathons/sessions/baby-steps/images/menuconfig_select_library.png)

We can then see all of Unikraft's internal libraries listed. Scroll down to `ukdebug`, and hit enter:

![ukdebug select1](/community/hackathons/sessions/baby-steps/images/menuconfig_select_ukdebug.png)

By default, only error messages are shown. We want to see all the messages (info, warnings, errors) logged by the kernel.

![ukdebug select2](/community/hackathons/sessions/baby-steps/images/menuconfig_select_ukdebug_kernel_message.png)

![ukdebug select3](/community/hackathons/sessions/baby-steps/images/menuconfig_select_ukdebug_kernel_message_all.png)

As before, press the `Save` button and exit the configuration menu by repeatedly selecting `Exit`.

Now, if we build and run the application in the same manner (i.e with `$ make -j $(nproc)`, followed by  `$ qemu-system-x86_64 -kernel build/helloworld_qemu-x86_64 -nographic`), we should see a more detailed output:

```console
$ qemu-system-x86_64 -kernel build/helloworld_qemu-x86_64 -nographic

SeaBIOS (version Arch Linux 1.16.2-1-1)


iPXE (http://ipxe.org) 00:03.0 C900 PCI2.10 PnP PMM+06FD3260+06F33260 C900



Booting from ROM..[    0.000000] Info: [libkvmplat] <setup.c @  245> Memory 00fd00000000-010000000000 outside mapped area
[    0.000000] Info: [libkvmplat] <bootinfo.c @   56> Unikraft Atlas (0.13.1~5eb820bd)
[    0.000000] Info: [libkvmplat] <bootinfo.c @   59> Architecture: x86_64
[    0.000000] Info: [libkvmplat] <bootinfo.c @   62> Boot loader : qemu-multiboot
[    0.000000] Info: [libkvmplat] <bootinfo.c @   67> Command line: build/helloworld_qemu-x86_64
[    0.000000] Info: [libkvmplat] <bootinfo.c @   69> Boot memory map:
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  000000100000-00000011a000 00000001a000 r-x 0000000000100000 krnl
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  00000011a000-000000121000 000000007000 r-- 000000000011a000 krnl
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  000000121000-000000171000 000000050000 rw- 0000000000121000 krnl
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  0000001200a0-0000001200a0 000000000000 rw- 00000000001200a0 krnl
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  000000171000-00000017101d 00000000001d r-- 0000000000171000 cmdl
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  000000172000-000000173000 000000001000 rw- 0000000000172000 cmdl
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  000000173000-000000183000 000000010000 rw- 0000000000173000 stck
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  000000183000-000007fe0000 000007e5d000 rw- 0000000000183000
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  000007fe0000-000008000000 000000020000 r-- 0000000007fe0000 rsvd
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  0000fffc0000-000100000000 000000040000 r-- 00000000fffc0000 rsvd
[    0.000000] Info: [libkvmplat] <bootinfo.c @  101>  00fd00000000-010000000000 000300000000 r-- 000000fd00000000 rsvd
[    0.000000] Info: [libkvmplat] <setup.c @  374> Switch from bootstrap stack to stack @0x183000
[    0.000000] Info: [libukboot] <boot.c @  251> Unikraft constructor table at 0x120000 - 0x120018
[    0.000000] Info: [libukboot] <boot.c @  288> Initialize memory allocator...
[    0.000000] Info: [libukallocbbuddy] <bbuddy.c @  515> Initialize binary buddy allocator 183000
[    0.000000] Info: [libukboot] <boot.c @  313> Initialize IRQ subsystem...
[    0.000000] Info: [libukboot] <boot.c @  320> Initialize platform time...
[    0.000000] Info: [libkvmplat] <tscclock.c @  255> Calibrating TSC clock against i8254 timer
[    0.100032] Info: [libkvmplat] <tscclock.c @  276> Clock source: TSC, frequency estimate is 3001267500 Hz
[    0.100902] Info: [libukboot] <boot.c @  324> Initialize scheduling...
[    0.101515] Info: [libukschedcoop] <schedcoop.c @  257> Initializing cooperative scheduler
[    0.103719] Info: [libukboot] <boot.c @  339> Init Table @ 0x120018 - 0x120020
[    0.104629] Info: [libukbus] <bus.c @  134> Initialize bus handlers...
[    0.105286] Info: [libukbus] <bus.c @  136> Probe buses...
[    0.105945] Info: [libkvmpci] <pci_bus_x86.c @  158> PCI 00:00.00 (0600 8086:1237): <no driver>
[    0.106905] Info: [libkvmpci] <pci_bus_x86.c @  158> PCI 00:01.00 (0600 8086:7000): <no driver>
[    0.107746] Info: [libkvmpci] <pci_bus_x86.c @  158> PCI 00:02.00 (0300 1234:1111): <no driver>
[    0.108581] Info: [libkvmpci] <pci_bus_x86.c @  158> PCI 00:03.00 (0200 8086:100e): <no driver>
Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~5eb820bd
[    0.112301] Info: [libukboot] <boot.c @  369> Pre-init table at 0x120090 - 0x120090
[    0.113121] Info: [libukboot] <boot.c @  380> Constructor table at 0x120090 - 0x120090
[    0.114027] Info: [libukboot] <boot.c @  401> Calling main(1, ['build/helloworld_qemu-x86_64'])
Hello world!
Arguments:  "build/helloworld_qemu-x86_64"
[    0.115738] Info: [libukboot] <boot.c @  410> main returned 0, halting system
[    0.116550] Info: [libkvmplat] <shutdown.c @   35> Unikraft halted
```

You can notice that the debug logs prove to be quite useful: they include information such as timestamp, debugging level (warning, info, error etc.), internal library component (`libkvmpci`, `libukschedcoop` etc.) and the source file along with the line in it.
### Enabling tests

As an exercise, try to enable the `uktest` internal library from the configuration menu.
This should trigger the execution of a suite of tests for some of Unikraft's internal libraries during the booting of the unikernel image.
You should get an output similar to this one:

```console
$ qemu-system-x86_64 -kernel build/helloworld_qemu-x86_64 -nographic

SeaBIOS (version Arch Linux 1.16.2-1-1)


iPXE (http://ipxe.org) 00:03.0 C900 PCI2.10 PnP PMM+06FD3260+06F33260 C900


[...]


[    0.000000] Info: test: uktest_myself_testsuite->uktest_test_sanity
[    0.000000] Info:    expected `((void *) 0)` to be 0 and was 0 ....................................... [ PASSED ]
[    0.000000] Info:    expected `1` to not be 0 and was 0x1 ............................................ [ PASSED ]
[    0.000000] Info:    expected `0` to be 0 and was 0 .................................................. [ PASSED ]
[    0.000000] Info:    expected `1` to not be 0 and was 1 .............................................. [ PASSED ]
[    0.000000] Info:    expected `a` to be 0x182f3c and was 0x182f3c .................................... [ PASSED ]
[    0.000000] Info:    expected `a` at 0x182f3c to equal `b` at 0x182f3c ............................... [ PASSED ]
[    0.000000] Info:    expected `1` to be 1 and was 1 .................................................. [ PASSED ]
[    0.000000] Info:    expected `0` to not be 1 and was 0 .............................................. [ PASSED ]
[    0.000000] Info:    expected `1` to be greater than 0 and was 1 ..................................... [ PASSED ]
[    0.000000] Info:    expected `1` to be greater than or equal to 1 and was 1 ......................... [ PASSED ]
[    0.000000] Info:    expected `2` to be greater than or equal to 1 and was 2 ......................... [ PASSED ]
[    0.000000] Info:    expected `0` to be less than 1 and was 0 ........................................ [ PASSED ]
[    0.000000] Info:    expected `1` to be less than or equal to 1 and was 1 ............................ [ PASSED ]
[    0.000000] Info:    expected `1` to be less than or equal to 2 and was 1 ............................ [ PASSED ]
[    0.000000] Info: [libukboot] <boot.c @  288> Initialize memory allocator...


[...]

[    0.138087] Info: [libuktest] <test.c @   80> uktest:suites:     2 total
[    0.138689] Info: [libuktest] <test.c @   93> uktest:cases:      6 total
[    0.139226] Info: [libuktest] <test.c @  106> uktest:assertions: 24 total
Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~5eb820bd
[    0.142533] Info: [libukboot] <boot.c @  369> Pre-init table at 0x1220a8 - 0x1220a8
[    0.143355] Info: [libukboot] <boot.c @  380> Constructor table at 0x1220a8 - 0x1220a8
[    0.144165] Info: [libukboot] <boot.c @  401> Calling main(1, ['build/helloworld_qemu-x86_64'])
Hello world!
Arguments:  "build/helloworld_qemu-x86_64"
[    0.145824] Info: [libukboot] <boot.c @  410> main returned 0, halting system
[    0.146611] Info: [libkvmplat] <shutdown.c @   35> Unikraft halted
```
