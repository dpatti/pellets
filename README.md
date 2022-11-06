# pellets

```
pellets [<config>] [--aur-install <command>]
```

A declarative `pacman` wrapper for Arch Linux. The goal of pellets is to allow
you to tersely specify what packages you expect to be installed and keep your
system tidy by removing packages that aren't needed. It's also great for
bootstrapping a new system. Pellets doesn't prevent you from installing new
packages manually to try them out, but it will help you remember to keep a
transcript of those you want to stay around.

You start by specifying an annotated file of packages you want installed, like
this:

```
# Apps
google-chrome [aur]
vlc

# Tools
maim # used to take screenshots

# X11
xorg [group]
xmonad
```

Then, when you run `pellets`, it will look at the current explicitly-installed
packages, compare them to the list, and install, remove, or modify to make your
system match the config file.

```
$ pellets
install maim
remove scrot
mark-explicit xorg-xrandr

Proceed? [Y/n/dry]:
```

The specific actions it can take are:

| in config | install reason       | action                    |
|-----------|----------------------|---------------------------|
| yes       | not installed        | install                   |
| yes       | installed as dep     | mark-explicit             |
| yes       | explicitly installed | none                      |
| no        | not installed        | none                      |
| no        | installed as dep     | none                      |
| no        | explicitly installed | mark-dependency OR remove |

Marking a package as an explicit installation or dependency is mostly for
bookkeeping purposes. It uses this information to prompt to prune unused
dependency packages after the previous step is complete.

## Config File

You can specify a path to a config file on the command line. If none is
specified, `pellets` will look in the default location using [XDG Base Directory
specification][xdg] which is typically `~/.config/pellets/packages`. Note that
even when run with `sudo`, `~` will resolve to the calling user's `$HOME`
directory. If you are using `$XDG_CONFIG_HOME`, you must use `sudo -E` to
preserve the environment.

[xdg]: https://wiki.archlinux.org/title/XDG_Base_Directory

A config file is simply a list of packages and comments. Comments begin with a
`#` and can appear at any point in the line. Packages are specified one per
line. There are two modifiers for packages:

* `[aur]` indicates that this package is built via the Arch User Repository
  (see below)
* `[group]` indicates that this is a package group

You may also specify a provision instead of a package in the same way you could
for pacman. This means that "python-neovim" and "python-pynvim" would both match
against the `python-pynvim` package which *provides* "python-neovim".

Provision and group resolution happens on each run. This means that if a group
was installed and a package was later added to the group, that new package will
be installed on the next run of pellets. This is *inconsistent* with how pacman
works -- pacman only remembers the actual packages that were installed, not by
what method they were installed.

If a package is not explicitly listed and only *optionally* depended on by other
packages, it will be removed. You should add it to your config if you wish to
keep it.

### Starting out

If you're starting from a blank slate, you can query what is currently installed
on your system:

```
$ pacman -Q --quiet --explicit --native && pacman -Q --quiet --explicit --foreign | xargs printf "%s [aur]\n"
```

Many of these packages will come from groups. If you want to learn about what
groups you might have installed, use `pacman -Q --explicit --groups` to get a
hint. The most common groups are `base-devel` and `xorg`.

## AUR Packages

By default, AUR packages are not installed since this isn't intended to be an
entire toolchain for building AUR packages. Instead, you can optionally pass a
command prefix with `--aur-install` that accepts packages on the command line.
For example:

```
pellets --aur-install "aura -A"
```

Alternatively, you can place an executable file at the path
`$XDG_CONFIG_HOME/pellets/aur-install` (where `$XDG_CONFIG_HOME` defaults to
`~/.config`). If `--aur-install` is not specified and the file is present, it
will be used. For example:

```
#!/usr/bin/env bash

exec aura -A --unsuppress --noconfirm "$@"
```

## Installation

<https://aur.archlinux.org/packages/pellets>
