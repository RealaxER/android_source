# KERNEL MODULE RPI4
***
## 1.BUILD IN DRIVER HELLO 
```
cd kernel/common/drivers
mkdir -p hello
```

## IMPLEMENT MAKEFILE
```
code hello/Makefile
```

### Makefile
```
obj-$(CONFIG_HELLO) += hello.o
```

## IMPLEMENT KCONFIG
```
code hello/Kconfig
```
### Kconfig
```
# Device Hello configuration
# By BHien 
menuconfig HELLO
    tristate "Hello String"
    default y
    help
      Say Y to include this module
      Say N will not build this module
      Say M to build this module but not include to kernel yet
```

## ADD MAKFILE AND KCONFIG TO DRIVER
```
code Makefile 
code Kconfig
```
### Makefile
```
obj-$(CONFIG_HELLO) += hello/
```
### Kconfig
```
source "drivers/hello/Kconfig"
```

## IMPLEMENT SOURCE
```
code hello/hello.c
```
### hello.c
```
#include <linux/module.h>	 /* Needed by all modules */
#include <linux/kernel.h>	 /* Needed for KERN_INFO */
#include <linux/init.h>	 /* Needed for the macros */

///< The license type -- this affects runtime behavior
MODULE_LICENSE("GPL");

///< The author -- visible when you use modinfo
MODULE_AUTHOR("Akshat Sinha");

///< The description -- see modinfo
MODULE_DESCRIPTION("A simple Hello world LKM!");

static int __init hello_start(void)
{
	printk(KERN_INFO "Loading hello module...\n");
	printk(KERN_INFO "Hello world\n");
	return 0;
}

static void __exit hello_end(void)
{
	printk(KERN_INFO "Goodbye Mr.\n");
}

module_init(hello_start);
module_exit(hello_end);
```

## BUILD WITH Config.sh
```
cd ../../
```
### :~/kernel$ 
```
BUILD_CONFIG=common/build.config.rpi4 \
LTO=none \
FAST_BUILD=1 \
SKIP_MRPROPER=1 \
SKIP_DEFCONFIG=1 \
build/config.sh
```
***
## 2. BUILD LOADABLE KERNEL MODULE
```
code common/build.config.rpi4
```
### ADD MODULES TO MAKE_GOALS
```
MAKE_GOALS="....
....
modules
"
```
### CREATE FOLDER HELLO
```
cd && mkdir -p hello
cd hello
```

### IMPLEMENT SOURCE 
```
code hello.c
```
### IMPLEMENT MAKEFILE
```
obj-m += hello.o
PWD := $(shell pwd)
CLANG := /home/asus/kernel/prebuilts/clang/host/linux-x86/clang-r450784e/bin/clang
KERNEL := /home/asus/kernel/out/common/
LD = /home/asus/kernel/prebuilts/clang/host/linux-x86/clang-r450784e/bin/ld.lld
all:
	make LD=$(LD) ARCH=arm64 CC=$(CLANG) -C $(KERNEL) M=$(PWD) modules

clean:
	make -C $(KERNEL) M=$(PWD) clean

```
### MAKE
```
make
```

## 3. ADB TEST MODULE
### ADB CONNECT 
```
adb root
adb devices
adb push hello.ko /dev  
```

### INSMOD MODULE

```
insmod hello.ko
dmesg | tail
rmmod hello.ko
```


## 4.BUILD KERNEL WITH BAZEL 
### CONFIGURATION KERNEL RPI4 
```
cd kernel/
code common/BUILD.bazel
code common/build.config.rpi4
```
### build.config.rpi4
```
MAKE_GOALS="${MAKE_GOALS}
dtbs
modules
Image.gz
"

FILES="${FILES}
arch/arm64/boot/Image.gz
arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb
arch/arm64/boot/dts/broadcom/bcm2711-rpi-400.dtb
arch/arm64/boot/dts/broadcom/bcm2711-rpi-cm4.dtb
arch/arm64/boot/dts/broadcom/bcm2711-rpi-cm4s.dtb
"
```
### BUILD.bazel
```
load("//build/kernel/kleaf:common_kernels.bzl", "define_common_kernels","define_rpi4")

...
# Sync with build.config.rpi4
define_rpi4(
    name = "rpi4",
    outs = aarch64_gz_outs + [
        "arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb",
        "arch/arm64/boot/dts/broadcom/bcm2711-rpi-400.dtb",
        "arch/arm64/boot/dts/broadcom/bcm2711-rpi-cm4.dtb",
        "arch/arm64/boot/dts/broadcom/bcm2711-rpi-cm4s.dtb",
    ],
    module_outs = [
    ],
)
```
### :~/kernel$
### CHECK RPI4 HAS BEEN ADDED
```
tools/bazel query "kind('py_binary', //common:*)"
```
### BUILD KERNEL WITH BAZEL 
```
tools/bazel run --config=fast //common:rpi4_dist
```
### IMAGE WAS CREATED AT 

```
cd out/android13-5.15/dist && ls
```

