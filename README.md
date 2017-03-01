# clyde

clyde is meant to be used in addition to [cpcache](https://github.com/nroi/cpcache), although it should work with
other pacman caches as well. Its purpose is to keep the cache warm, i.e., to download packages from the cache such that subsequent
HTTP GET requests of the same package can be fulfilled without having to download the file from a remote mirror.

## Installation
A package is available in [AUR](https://aur.archlinux.org/packages/clyde-git/).
After installation, edit `/etc/clyde/config.yml` if necessary (no changes are required if the default settings of cpcache have
not been changed).

Make sure you have some files (with arbitrary file names) listed in the `/etc/clyde/wanted_packages`
directory.
Each line in each file inside this directory will be downloaded by clyde, unless the package is
already cached. The intention behind using multiple files is that you can use one file for each
client that makes use of cpcache.
Instead of creating those files manually, you can also send a POST request to cpcache, for instance
(replace `127.0.0.1:7070` by the actual address of cpcache):
```bash
pacman -Qq | curl --data-binary @- "http://127.0.0.1:7070/$(hostname)" -H 'Content-Type:text/plain; charset=utf8'
```
This will send the currently installed packages to cpcache, which will save them in a file named after
your host.

Once you have done this, start and enable the clyde timer:
```bash
systemctl start clyde.timer
systemctl enable clyde.timer
```

clyde will now run each night at around 4 o'clock (± one hour) to fetch all packages listed in `/etc/clyde/wanted_packages` if
a version that is more recent than the one residing on the local filesystem is available.
Note that clyde itself does not store the packages (all files are downloaded to /dev/null), since cpcache will already store
all files locally.
