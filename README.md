# p44build

## Introduction

**p44build** (or **p44b** for short) is a script that helps managing differently configured OpenWrt targets on top of an unpolluted OpenWrt original tree.

The basic idea is that a OpenWrt based firmware project, including everything needed to exactly reproduce it, can itself be stored in a "umbrella" package
and managed/versioned along with other packages in a custom feed.

All p44b specific information is stored in a folder called "p44build", usually a subfolder of the "umbrella" openwrt package.

This can include a set of global patches that need to be applied to the openwrt tree (openwrt's quilt tool is used for that),
configurations (diffconfigs) for one or multiple target platforms the project can be built for (my own projects often have versions for Omega2 and RaspberryPi), and custom scripts for project specific build, package, flash and deploy steps.

The p44build folder also has a "tracking" subfolder, which collects information about the exact state of all openwrt feeds, source packages installed, and can additionally track the individual state of packages
that are part of the development for this target.

So with a simple `./p44b save` command, everything needed to reproduce that build on top of a totally fresh openwrt checkout
is recorded into the umbrella package and can be versioned easily by a commit in the umbrella package's feed.

## Basic usage

Start with a fresh openwrt checkout, and an umbrella openwrt packge. Look at one of the *-config packages in the
[plan44 feed](https://github.com/plan44/plan44-feed), like [pixelboard-config](https://github.com/plan44/plan44-feed/tree/master/pixelboard-config) which I'll use as example below.
(Alternatively, use the `new` command to create a default p44build directory with the current openwrt version and config as a starting point for a new project.)

The openwrt feed should be checked out at the version required by the project, otherwise `./p44b init` will complain. The file `buildroot_checkout` in the p44build folder (`feeds/plan44/pixelboard-config/p44build` in this example) contains the version label expected.

You also need this repo checked out somewhere, let me assume it is in `/users/you/stuff/p44build`

Now from here the steps to build an image are

- `cd /users/you/openwrt` # change into the root of your openwrt checkout
- `/users/you/stuff/p44build/p44b init feeds/plan44/pixelboard-config/p44build` # this initializes p44b and puts a soft link to itself into the openwrt root, so we can use `./p44b` from here on
- `./p44b prepare` # this applies global patches to the openwrt tree (which can be removed again by `./p44b cleanup`)
- `./p44b target` # to see what target platforms the project can build for
- `./p44b target pixelboard-omega2` # select the Omega2 target, apply openwrt config
- `./p44b build` # build the image
- `TARGET_HOST=192.168.xx.yy ./p44b send` # send the built image to /tmp on TARGET_HOST, where it can be installed using `sysupgrade /tmp/*sysupgrade.bin`

At this point, you can also actively work in the tree such as using `make menuconfig` to change openwrt config, install additional packages, continue developing the main application ([pixelboard](https://github.com/plan44/pixelboardd) in this case) etc.

To save a new build state back into the "umbrella" package just use `./p44b save`.

To revert the openwrt tree back to clean checkout state: `./p44b cleanup`

## Advanced features

p44b has many other subcommands (just use `./p44b --help` or `./p44b <subcommand> --help` for packaging, debugging, deploying, tagging involved git projects etc. Some of these need helper scripts in the p44build folder that do the actual work as needed for the project.

## Questions, Suggestions, Contributions

- There is a forum at [forum.plan44.ch](https://forum.plan44.ch/t/opensource-c-vdcd) for all things related to p44 opensource, best for questions about `p44b`.
- Of course, issues or pull requests on github are possible.
