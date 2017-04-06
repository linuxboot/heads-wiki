Building Heads
===
Heads is supposed to be a [reproducible build](https://reproducible-builds.org/) and as of [v0.1.0](https://github.com/osresearch/heads/releases/tag/v0.1.0) it achieved this goal.  The downside is that the initial build can take a very long time as it downloads and builds all of the its dependencies.  One issue right now is that it builds not just one, but *two* cross compilers and as a result takes about 45 minutes.  Luckily subsequent builds only take about 30 seconds to produce a full coreboot and Linux ROM image, but that first ones a doozy...

With a vanilla Ubuntu 16.04 install, such as a digitalocean droplet, you need to first install some support tools. This takes a short while, so get a cup of coffee:

```
apt update
apt install build-essential
apt install flex bison
```

Clone the tree:

```
git clone https://github.com/osresearch/heads
cd heads
```

Run `make` and it will start the downloads and building process.  This takes a long while, so go out for a cup of coffee..

On a small 1GB machine it is not possible to use the `-j8` option to musl-cross (specified in `build/musl-cross-git/config.sh` as `MAKEFLAGS`).  `-j1` is necessary to avoid running out of virtual memory (todo: allow this to be passed via `MAKEJOBS` on the command line or derive it from `nproc`?)
