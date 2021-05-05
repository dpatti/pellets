# pellets

A declarative `pacman` wrapper for Arch Linux. Allows you to specify an
annotated config file, like this:

```
# Apps
google-chrome [aur]
vlc

# Tools
base-devel [group]
maim # used for screenshots

# X11
xorg [group]
xmonad
```

Then, when you run `pellets path/to/config`, it will look at the current
explicitly-installed packages, compare them to the list, and install, remove, or
modify to make your system match the config file. The specific actions it can
take are:

| in config | install reason       | action                       |
|-----------|----------------------|------------------------------|
| yes       | not installed        | install                      |
| yes       | installed as dep     | mark as explicit             |
| yes       | explicitly installed | none                         |
| no        | not installed        | none                         |
| no        | installed as dep     | none                         |
| no        | explicitly installed | mark as dependency OR remove |

## Notes

If a package is not explicitly required and only *optionally* depended on by
other packages, it will be removed. You should add it to your config if you wish
to keep it.

It doesn't actually do anything yet. It just prints what it would do if it were
finished. It's probably like 5 more lines of code that I haven't written. Except
the AUR part, that's maybe less straightforward.

You can specify packages *or* provisions in the same way you could for pacman.
This means that "python-neovim" and "python-pynvim" in the config would both
match against the `python-pynvim` package which *provides* "python-neovim".

Provision and group resolution happens on each run. This means that if a group
was installed and a package was later added to the group, that new package will
be installed on the next run of pellets. This is *inconsistent* with how pacman
works -- pacman only remembers the actual packages that were installed, not by
what method they were installed.

It's written in bash and will probably just break all the time.
