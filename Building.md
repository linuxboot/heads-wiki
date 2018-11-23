Building Heads
===
Heads is supposed to be a [reproducible build](https://reproducible-builds.org/) and as of [v0.1.0](https://github.com/osresearch/heads/releases/tag/v0.1.0) it achieved this goal.  The downside is that the initial build can take a very long time as it downloads and builds all of the its dependencies.  One issue right now is that it builds not just one, but *two* cross compilers and as a result takes about 45 minutes.  Luckily subsequent builds only take about 30 seconds to produce a full coreboot and Linux ROM image, but that first ones a doozy...

With a vanilla Debian 9 or Ubuntu 16.04 install, such as a digitalocean
droplet, you need to first install some support tools. This takes a
short while, so get a cup of coffee:

```
apt update
apt install -y \
	build-essential \
	zlib1g-dev uuid-dev libdigest-sha-perl \
	libelf-dev \
	bc \
	bzip2 \
	bison \
	flex \
	git \
	gnupg \
	iasl \
	m4 \
	nasm \
	patch \
	python \
	wget \
	gnat \
	cpio \
	ccache
```

On a Fedora machine:
```
dnf install -y \
	@development-tools \
	gcc-c++ zlib-devel perl-Digest-MD5 perl-Digest-SHA \
	uuid-devel pcsc-tools qemu ncurses-devel lbzip2 \
	libuuid-devel lzma \
	elfutils-libelf-devel \
	bc \
	bzip2 \
	bison \
	flex \
	git \
	gnupg \
	iasl \
	m4 \
	nasm \
	patch \
	python \
	wget \
	libusb-devel \
	cmake

# For emulation and analysis with UEFITool
dnf install -y \
	qemu qt5-devel \
```

Clone the tree:

```
git clone https://github.com/osresearch/heads
cd heads
```

Run `make` and it will start the downloads and building process for a qemu
emulated Heads+coreboot ROM image.  This takes a long while, so go out
for a cup of coffee..  The initial build on a small 1-core 1GB droplet
it will take over 90 minutes, an 8-core system takes about 40 minutes.

Useful targets, stored under the `board` directory of the git tree. 

Generated roms are generally found under build/$BOARD/$BOARD.rom

---
Generic
---
Generally, everything that is needed to flash the SPI flash of a board is a single rom generated through `make BOARD=$BOARD` command, where $BOARD is the name of the board that can be found under `board` directory of the git downloaded tree.


```
make BOARD=kgpe-d16
```
will produce a build/kgpe-d16/kgpe-d6.rom

---
Make for a specific configuration. 
---
Some boards have a two SPI flash chip configuration and need special care.

----
x230 and x220
----

-----
Initial SPI2 (4MB) flash chips
-----
x230 and x220 boards needs their 4MB SPI2 to be initially externally flashed, while the 12MB rom needs to be flashed internally from within Heads to make sure to not screw up with ME, contained in the SPI1 flash (8MB bottom flash chip under keyboad)

The following make command generates a self-contained, externally flashable rom for the SPI2 (4MB BIOS, top SPI flash under keyboard). 


```
make BOARD=x230-flash
```
Resulting rom is found under build/x230-flash/x230-flash.rom

-----
Subsequent flashing (upgrades)
-----

The following make command will generate 3 rom images under build/x230 directory: a 12MB `coreboot.rom`, a `x230-4.rom` and a `x230-8.rom`.

```
make BOARD=x230
```

The `coreboot.rom` is the one needed to flash rom updates from within Heads in respect of ME. This is done with the help of the `flashrom-x230.sh` script from Heads recovery shell.

More information under [`Installing Heads`](https://github.com/osresearch/heads-wiki/blob/master/Installing-Heads.md) and [`Cleaning the ME firmware`](https://github.com/osresearch/heads-wiki/blob/master/Clean-the-ME-firmware.md)

---
Helpful targets and options
---

Verbose build (otherwise all log output goes into `build/logs/$(submodule).log`):
```
make V=1
```

Produce just the build of a single sub-module with the `.intermediate` suffix:
```
make gpg.intermediate
```

Clean a single submodule or all (?) volatile submodules:
```
make gpg.clean
make modules.clean
```


The Heads `Makefile`
===
All of the organization of the Heads build is handled in the top level `Makefile` with the goal of producing a reproducible `initrd.cpio` containing the Heads runtime and kernel modules, the Head's Linux `bzImage` kernel, and the `coreboot.rom` tailored for the target platform initialization.

Build configuration
---
Platform configuration are stored in the `board/$BOARD.config`
(this might change); these files specify the mainboard (`x230`, `x230-flash`
`chell`, `librem13v1`, and servers like `s2600wf`, `winterfell`, `kgpe-d16` and `r630`)
as well as the sub-modules necessary for the system.
The main difference between these use cases is the init scripts that
are installed in the inird, the Linux kernel configuration and the
coreboot or edk2 configuration.
An example configuration is [`board/x230.config`](https://github.com/osresearch/heads/blob/master/boards/x230.config)


Sub-modules
---
Sub-modules are defined in the `modules` directory and each one defines a dependency to be fetched, verified, configured, built and installed.  The top level `Makefile` includes all of the `modules/*` files and the ones that are configured to `y` in the board's configuration will be built and installed into the initrd.  Since the Heads runtime is built with a `musl-libc` cross compiler, there are frequently hacks necessary to convince the `configure` scripts or submodule Makefiles to build correctly, and since we only want the bare minimum of output for the initrd we don't use the actual `make install` target to create the ramdisk.

An example submodule file is for the `cryptsetup` command, used to mount encrypted volumes from the recovery shell:

```
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
	--host i386-elf-linux \
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
* Give itself a name and add itself to the modules list, if configured: `modules-$(CONFIG_CRYPTSETUP) += cryptsetup`
* Enumerate its depdencies by their submodule names: `cryptsetup_depends := util-linux popt lvm2 $(musl_dep)`
* Define a version `cryptsetup_version := 1.73`
* Specify the name of the directory that the tar file will unpack into `cryptsetup_dir`
* Name the tar file that will be fetched `cryptsetup_tar`
* Specify the URL from which this file will be fetched `cryptsetup_url`
* Provide the sha256 hash of the file to verify its correctness `cryptsetup_hash`
* Define the command that will be run in the unpacked diretory to configure it after unpacking and patching (see below) `cryptsetup_configure`
** Note that we provide the compiler `CC="$(heads_cc)"` in quotes, since there might be spaces in the variable
  * `--host i386-elf-linux` is used to indicate that this is a cross compile
  * `--prefix` avoids writing path names into the executable
  * plus some additional submodule specific stuff
* The target to be called by make in this directory to generate the output
  * The `$(MAKE_JOBS)` will have the number of parallel instances to execute
  * Sometimes mutliple steps are required; separate them by `&&` to ensure that they short-circuit correctly
  * The first one has an implicit `$(MAKE) -C $(build)/$(cryptsetup_dir)`, but the second step has to list it explicitly
* Lastly a list of any executables `cryptsetup_output` or libraries `cryptsetup_libraries` that will be produced by the build, relative to the build directory.
  * These will be installed into `initrd/bin` and `initrd/lib`
  * They will be stripped

If there are any patches that need to be applied, put the file in `patches/$(submodule_name)-$(submodule_version).patch` and it will be applied when the tar file is unpacked.
