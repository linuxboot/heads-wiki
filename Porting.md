To add a new board to the Heads build hopefully you only need to modify
the coreboot configuration and add a few targets to the top level
`Makefile`.

* Start by copying one of the existing config files, such as the
`config/coreboot-x230.config` to `config/coreboot-newarch.conf`.

* Add a target in the `Makefile` to convert the `coreboot.rom`
file to your `newarch.rom`.  For qemu-like targets the entire
16MB coreboot output can be used, but for real systems like the
x230 it is necessary to slice out just a small piece.  For ones
like the chell chromebook it is necessary to bundle it with the ME
and other blobs as well.

```
x230.rom: $(build)/$(coreboot_dir)/x230/coreboot.rom
        "$(build)/$(coreboot_dir)/$(BOARD)/cbfstool" "$<" print
        dd if="$<" of="$@" bs=1M skip=8
        $(RM) "$<"
        @sha256sum "$@"
```

* Run `make BOARD=newarch` to setup the coreboot tree, using the new
coreboot config file.  This will create the output directory
`build/coreboot-4.5/newarch/`.

* If you want to use `menuconfig` to reconfigure the coreboot file,
it is a little tricky since we have an external output directory and
config file.  Run:

```
make \
	-C build/coreboot-4.5 \
	obj=./newarch \
	DOTCONFIG=../../config/coreboot-newarch.config \
	menuconfig
```

* If things don't work, please open an issue on https://github.com/osresearch/heads/issues
