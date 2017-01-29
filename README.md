# clyde

clyde is meant to be used in addition to [cpcache](https://github.com/nroi/cpcache), although it should work with
other pacman caches as well. Its purpose is to keep the cache warm, i.e., to download packages from the cache such that subsequent
HTTP GET requests of the same package can be fulfilled without having to download the file from a remote mirror.

## Installation
A package is available in [AUR](https://aur.archlinux.org/packages/clyde-git/).
After installation, edit `/etc/clyde/config.yml` if necessary (no changes are required if the default settings of cpcache have
not been changed). 

Make sure you have some packages listed in `/etc/clyde/wanted_packages`. For instance, if you want clyde to regularly download
the most recent version version of all packages that you have currently installed, run
```bash
pacman -Q | cut -f1 --delimiter=" " > wanted_packages
```
on the machine that is going to fetch packages using cpcache. Then, copy the resulting file to
the `/etc/clyde/wanted_packages` directory residing on the machine running cpcache.

Once you have done this, start and enable the clyde timer:
```bash
systemctl start clyde.timer
systemctl enable clyde.timer
```

clyde will now run each night at around 4 o'clock (± one hour) to fetch all packages listed in `/etc/clyde/wanted_packages` if
a version that is more recent than the one residing on the local filesystem is available.
Note that clyde itself does not store the packages (all files are downloaded to /dev/null), since cpcache will already store
all files locally.
