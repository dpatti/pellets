# pellets

A declarative `pacman` wrapper for Arch Linux. Pellets allows you to specify an
annotated config file, like this:

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

## AUR Packages

By default, AUR packages are not installed since this isn't intended to be an
entire toolchain for building AUR packages. Instead, you can optionally pass a
command prefix with `--aur-install` that accepts packages on the command line.
For example:

```
pellets --aur-install "aura -A"
```

## Notes

If a package is not explicitly required and only *optionally* depended on by
other packages, it will be removed. You should add it to your config if you wish
to keep it.

You can specify packages *or* provisions in the same way you could for pacman.
This means that "python-neovim" and "python-pynvim" in the config would both
match against the `python-pynvim` package which *provides* "python-neovim".

Provision and group resolution happens on each run. This means that if a group
was installed and a package was later added to the group, that new package will
be installed on the next run of pellets. This is *inconsistent* with how pacman
works -- pacman only remembers the actual packages that were installed, not by
what method they were installed.

It's written in bash and will probably just break all the time.
