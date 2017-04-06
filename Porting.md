To add a new board to the Heads build hopefully you only need to modify
the coreboot configuration and add a top-level image configuration.

* Copy one of the existing image configurations, such as `config/x230-qubes.config`
to your new image, `config/newarch-qubes.config`, and edit the file to change
`BOARD=x230` to `BOARD=newarch`.

* Either copy one of the existing config files, such as the
`config/coreboot-x230.config` to `config/coreboot-newarch.conf`,
or use your existing coreboot `.config` file in its place.  You'll want to
be sure that it is using a Linux payload named `./bzImage` and a Linux initrd
named `./initrd.cpio.xz`.  No command line options are necessary.

* If you want to use `menuconfig` to reconfigure the coreboot file,
it is a little tricky since we have an external config file.  From the
top level of the Heads directory you can run:

```
make \
	-C build/coreboot-4.5 \
	DOTCONFIG=../../config/coreboot-newarch.config \
	menuconfig
```

* Run `make CONFIG=config/newarch-qubes.config` to setup the coreboot tree,
using the new coreboot config file.  This will create the output directory
`build/coreboot-4.5/newarch/`, and after a while, you should have `newarch.rom`
in the top level directory.

* If things don't work, please open an issue on https://github.com/osresearch/heads/issues
