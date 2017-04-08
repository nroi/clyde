# clyde

clyde is meant to be used in addition to [cpcache](https://github.com/nroi/cpcache).
Its purpose is to keep the cache warm, i.e., to download packages from the cache such that subsequent
HTTP GET requests of the same package can be fulfilled without having to download the file from a remote mirror.

## Installation
The project is split into
[clyde-client](https://aur.archlinux.org/packages/clyde-client-git/)
and
[clyde-server](https://aur.archlinux.org/packages/clyde-server-git/).
clyde-server is meant to be installed on the machine where cpcache is installed, clyde-client
should be installed on all devices which use cpcache in order to install packages.

clyde-client uses HTTP POST requests to send a list of required packages to cpcache. These POST
requests need to be authorized, so we need to choose a key first. The same key will be used by all
clients and the server. The key is sent along in an HTTP header, so avoid using illegal characters
(`openssl rand -hex 25` can be used to generate valid passwords).
Enter the key in `/etc/clyde_client/key` on each client, for instance:
```bash
echo "topsecret" > /etc/clyde_client/key
```
Then, enter the same key in `/etc/cpcache/cpcache.yaml` on the server, where cpcache is installed:
```yaml
recv_packages:
    key: "topsecret"
```

Start and enable `clyde_client.service` on all clients:
```bash
systemctl start clyde_client.service
systemctl enable clyde_client.service
```

This service will wait for pacman to access its database. As soon as pacman exits after
having accessed the database, a list containing all currently installed packages is sent to cpcache.
clyde-client looks for the address of cpcache in `/etc/pacman.d/mirrorlist`,
so make sure you have a line with a trailing `!cpcache` comment, like this:
```bash
Server = http://alarm.local/$repo/os/$arch # !cpcache
```

On the server, after having installed clyde-server, start and enable `clyde.timer`:
```bash
systemctl start clyde_server.timer
systemctl enable clyde_server.timer
```

clyde-server will now run each night at around 4 o'clock (± one hour). For each package used by at
least one client, it will fetch the most recent version, unless already present.
