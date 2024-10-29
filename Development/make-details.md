---
layout: default
title: Makefile
permalink: /Makefile/
nav_order: 2
parent: Development
---

Makefile
===

Helpful targets and options
---

Verbose build (otherwise all log output goes into `build/x86/logs/$(submodule).log`):

```shell
make V=1
```

Produce just the build of a single sub-module invoking its name as the target:

```shell
make gpg
```

Clean a single submodule or all (?) volatile submodules:

```shell
make gpg.clean
make modules.clean
```

Clean all volatile submodules except crosscompiler (clean build):

```shell
make real.clean
```

The Heads `Makefile`
===

All of the organization of the Heads build is handled in the top level
 `Makefile` with the goal of producing a reproducible `initrd.cpio` containing
 the Heads runtime and kernel modules, the Head's Linux `bzImage` kernel, and
 the `coreboot.rom` tailored for the target platform initialization.

Build configuration
---

Platform configuration are stored in the `board/$BOARD.config`
 ([list of boards can be found here](/Install-and-Configure#supported-devices))
as well as the sub-modules necessary for the system.
The main difference between these use cases is the init scripts that
are installed in the inird, the Linux kernel configuration and the
coreboot or edk2 configuration.
An example configuration is [`board/x230.config`](https://github.com/osresearch/heads/blob/master/boards/x230/x230.config)

Sub-modules
---

Sub-modules are defined in the `modules` directory and each one defines a
 dependency to be fetched, verified, configured, built and installed.  The top
 level `Makefile` includes all of the `modules/*` files and the ones that are
 configured to `y` in the board's configuration will be built and installed into
 the initrd.  Since the Heads runtime is built with a `musl-libc` cross
 compiler, there are frequently hacks necessary to convince the `configure`
 scripts or submodule Makefiles to build correctly, and since we only want the
 bare minimum of output for the initrd we don't use the actual `make install`
 target to create the ramdisk.

An example submodule file is for the `cryptsetup` command, used to mount
 encrypted volumes from the recovery shell:

```Makefile
modules-$(CONFIG_CRYPTSETUP) += cryptsetup

cryptsetup_depends := util-linux popt lvm2 $(musl_dep)

cryptsetup_version := 1.7.3
cryptsetup_dir := cryptsetup-$(cryptsetup_version)
cryptsetup_tar := cryptsetup-$(cryptsetup_version).tar.xz
cryptsetup_url := https://www.kernel.org/pub/linux/utils/cryptsetup/v1.7/cryptsetup-$(cryptsetup_version).tar.xz
cryptsetup_hash := af2b04e8475cf40b8d9ffd97a1acfa73aa787c890430afd89804fb544d6adc02

# Use an empty prefix so that the executables will not include the
# build path.
cryptsetup_configure := ./configure \
  CC="$(heads_cc)" \
  --host $(MUSL_ARCH)-elf-linux \
  --prefix "" \
  --disable-gcrypt-pbkdf2 \
  --with-crypto_backend=kernel \

# but after building, replace prefix so that they will be installed
# in the correct directory.
cryptsetup_target := \
  $(MAKE_JOBS) \
  && $(MAKE) \
    -C $(build)/$(cryptsetup_dir) \
    prefix="$(INSTALL)" \
    install

cryptsetup_output := \
  src/.libs/cryptsetup \
  src/.libs/veritysetup \

cryptsetup_libraries := \
  lib/.libs/libcryptsetup.so.4 \
```

The main components that every submodule must define are:

* Give itself a name and add itself to the modules list, if configured:
 `modules-$(CONFIG_CRYPTSETUP) += cryptsetup`
* Enumerate its depdencies by their submodule names:
 `cryptsetup_depends := util-linux popt lvm2 $(musl_dep)`
* Define a version `cryptsetup_version := 1.73`
* Specify the name of the directory that the tar file will unpack into `cryptsetup_dir`
* Name the tar file that will be fetched `cryptsetup_tar`
* Specify the URL from which this file will be fetched `cryptsetup_url`
* Provide the sha256 hash of the file to verify its correctness `cryptsetup_hash`
* Define the command that will be run in the unpacked diretory to configure it
 after unpacking and patching (see below) `cryptsetup_configure`
** Note that we provide the compiler `CC="$(heads_cc)"` in quotes, since there
 might be spaces in the variable
  * `--host $(MUSL_ARCH)-elf-linux` is used to indicate that this is a cross compile
  * `--prefix` avoids writing path names into the executable
  * plus some additional submodule specific stuff
* The target to be called by make in this directory to generate the output
  * The `$(MAKE_JOBS)` will have the number of parallel instances to execute
  * Sometimes mutliple steps are required; separate them by `&&` to ensure that
 they short-circuit correctly
  * The first one has an implicit `$(MAKE) -C $(build)/$(cryptsetup_dir)`, but
 the second step has to list it explicitly
* Lastly a list of any executables `cryptsetup_output` or libraries
 `cryptsetup_libraries` that will be produced by the build, relative to the
 build directory.
  * These will be installed into `initrd/bin` and `initrd/lib`
  * They will be stripped

If there are any patches that need to be applied, put the file in
 `patches/$(submodule_name)-$(submodule_version).patch` and it will be applied
 when the tar file is unpacked.
