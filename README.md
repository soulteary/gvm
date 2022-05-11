# gvm

GVM provides an interface to manage Go versions.

The current maintainer of the fork is [@soulteary](https://github.com/soulteary), fixed some issues that were not exposed in the past few years, **and supports Apple M1 devices**.

- Thanks to the original author of the program: By Josh Bussdieker (jbuss, jaja, jbussdieker) while working at [Moovweb](https://www.moovweb.com)
- Thanks to previous project maintainers: [Benjamin Knigge](https://github.com/BenKnigge)


Features
========
* Install/Uninstall Go versions with `gvm install [tag]` where tag is "60.3", "go1", "weekly.2011-11-08", or "tip"
* List added/removed files in GOROOT with `gvm diff`
* Manage GOPATHs with `gvm pkgset [create/use/delete] [name]`. Use `--local` as `name` to manage repository under local path (`/path/to/repo/.gvm_local`).
* List latest release tags with `gvm listall`. Use `--all` to list weekly as well.
* Cache a clean copy of the latest Go source for multiple version installs.
* Link project directories into GOPATH

Background
==========
When we started developing in Go mismatched dependencies and API changes plauged our build process and made it extremely difficult to merge with other peoples changes.

After nuking my entire GOROOT several times and rebuilding I decided to come up with a tool to oversee the process. It eventually evolved into what gvm is today.

Installing
==========

To install:

```bash
curl -sSL https://github.com/soulteary/gvm/raw/master/binscripts/gvm-installer | bash
```

Or if you are using zsh just change `bash` with `zsh`.


It is recommended to manually add the following content to `~/.bashrc` or `~/.zshrc` , then restart Terminal and use the `gvm` command.


```bash
export GO111MODULE=on
# set mirroring, if you need to
export GOPROXY=https://goproxy.io,direct
# or `export GOPROXY="https://goproxy.cn"`
export GOPATH="$HOME/go"
PATH="$GOPATH/bin:$PATH"
# you can set as your own mirror
# export GO_BINARY_BASE_URL=https://golang.google.cn/dl/
[[ -s "$HOME/.gvm/scripts/gvm" ]] && source "$HOME/.gvm/scripts/gvm"
export GOROOT_BOOTSTRAP=$GOROOT
```

Installing Go
=============

```bash
# download golang binaries, "clean and hygienic"
gvm install go1.18 -B
# or download golang source code, complie and install
# gvm install go1.18
gvm use go1.18 [--default]
```

Once this is done Go will be in the path and ready to use.
`$GOROOT` and `$GOPATH` are set automatically.

Additional options can be specified when installing Go:

    Usage: gvm install [version] [options]
        -s,  --source=SOURCE      Install Go from specified source.
        -n,  --name=NAME          Override the default name for this version.
        -pb, --with-protobuf      Install Go protocol buffers.
        -b,  --with-build-tools   Install package build tools.
        -B,  --binary             Only install from binary.
             --prefer-binary      Attempt a binary install, falling back to source.
        -h,  --help               Display this message.
        
### A Note on Compiling Go 1.5+
Go 1.5+ removed the C compilers from the toolchain and [replaced][compiler_note] them with one written in Go. Obviously, this creates a bootstrapping problem if you don't already have a working Go install. In order to compile Go 1.5+, make sure Go 1.18 is installed first.

```bash
gvm install go1.18 -B
gvm use go1.18
export GOROOT_BOOTSTRAP=$GOROOT
gvm install go1.18
```

[compiler_note]: https://docs.google.com/document/d/1OaatvGhEAq7VseQ9kkavxKNAfepWy2yhPUBs96FGV28/edit

List Go Versions
================
To list all installed Go versions (The current version is prefixed with "=>"):

```bash
gvm list
```

To list all Go versions available for download:

```bash
gvm listall
```

Uninstalling
============
To completely remove gvm and all installed Go versions and packages:

```bash
gvm implode
```

If that doesn't work see the troubleshooting steps at the bottom of this page.

Mac OS X Requirements
====================
 * Install Mercurial from https://www.mercurial-scm.org/downloads
 * Install Xcode Command Line Tools from the App Store.

```bash
xcode-select --install
brew update
brew install mercurial
```

Linux Requirements
==================

Debian/Ubuntu
==================

```bash
sudo apt-get install curl git mercurial make binutils bison gcc build-essential
```

Redhat/Centos
==================

```bash
sudo yum install curl
sudo yum install git
sudo yum install make
sudo yum install bison
sudo yum install gcc
sudo yum install glibc-devel
```

 * Install Mercurial from http://pkgs.repoforge.org/mercurial/

FreeBSD Requirements
====================

```bash
sudo pkg_add -r bash
sudo pkg_add -r git
sudo pkg_add -r mercurial
```

Vendoring Native Code and Dependencies
==================================================
GVM supports vendoring package set-specific native code and related
dependencies, which is useful if you need to qualify a new configuration
or version of one of these dependencies against a last-known-good version
in an isolated manner.  Such behavior is critical to maintaining good release
engineering and production environment hygiene.

As a convenience matter, GVM will furnish the following environment variables to
aid in this manner if you want to decouple your work from what the operating
system provides:

1. ``${GVM_OVERLAY_PREFIX}`` functions in a manner akin to a root directory
  hierarchy suitable for auto{conf,make,tools} where it could be passed in
  to ``./configure --prefix=${GVM_OVERLAY_PREFIX}`` and not conflict with any
  existing operating system artifacts and hermetically be used by your
  workspace.  This is suitable to use with ``C{PP,XX}FLAGS and LDFLAGS``, but you will have
  to manage these yourself, since each tool that uses them is different.

2. ``${PATH}`` includes ``${GVM_OVERLAY_PREFIX}/bin`` so that any tools you
  manually install will reside there, available for you.

3. ``${LD_LIBRARY_PATH}`` includes ``${GVM_OVERLAY_PREFIX}/lib`` so that any
  runtime library searching can be fulfilled there on FreeBSD and Linux.

4. ``${DYLD_LIBRARY_PATH}`` includes ``${GVM_OVERLAY_PREFIX}/lib`` so that any
  runtime library searching can be fulfilled there on Mac OS X.

5. ``${PKG_CONFIG_PATH}`` includes ``${GVM_OVERLAY_PREFIX}/lib/pkgconfig`` so
  that ``pkg-config`` can automatically resolve any vendored dependencies.

Recipe for success:

```bash
gvm use go1.18
gvm pkgset use current-known-good
# Let's assume that this includes some C headers and native libraries, which
# Go's CGO facility wraps for us.  Let's assume that these native
# dependencies are at version V.
gvm pkgset create trial-next-version
# Let's assume that V+1 has come along and you want to safely trial it in
# your workspace.
gvm pkgset use trial-next-version
# Do your work here replicating current-known-good from above, but install
# V+1 into ${GVM_OVERLAY_PREFIX}.
```

See examples/native for a working example.

Troubleshooting
===============
Sometimes especially during upgrades the state of gvm's files can get mixed up. This is mostly true for upgrade from older version than 0.0.8. Changes are slowing down and a LTR is imminent. But for now `rm -rf ~/.gvm` will always remove gvm. Stay tuned!

[![Gitter](https://badges.gitter.im/GoVesionManager/community.svg)](https://gitter.im/GoVesionManager/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
