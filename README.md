# clyde

clyde is meant to be used in addition to [cpcache](https://github.com/nroi/cpcache).
Its purpose is to keep the cache warm, i.e., to download packages from the cache such that subsequent
HTTP GET requests of the same package can be fulfilled without having to download the file from a remote mirror.
Before installing it, make sure you have setup cpcache. In particular, your `/etc/pacman.d/mirrorlist` file needs a line with a trailing `!cpcache` comment, such has:
```bash
Server = http://alarm.local:7070/$repo/os/$arch # !cpcache
```
This comment is required so that clyde_client knows how to reach cpcache.

## Installation
The project is split into
[clyde-client](https://aur.archlinux.org/packages/clyde-client-git/)
and
[clyde-server](https://aur.archlinux.org/packages/clyde-server-git/).
clyde-server is meant to be installed on the machine where cpcache is installed, clyde-client
should be installed on all devices which use cpcache in order to install packages.

The line with the trailing `!cpcache` comment is used so that clyde-client knows how to reach cpcache. After clyde-client has been installed, a pacman hook is run whenever a new package has been installed or removed. This hook will run `/usr/bin/clyde_client`, which will send a POST request to cpcache.
These POST requests need to be authorized, so we need to choose a key first. The same key will be used by all
clients and the server. The key is sent along in an HTTP header, so avoid using illegal characters
(`openssl rand -hex 25` can be used to generate valid passwords).
Enter the key in `/etc/clyde_client/key` on each client, for instance:
```bash
echo "topsecret" > /etc/clyde_client/key
```

Then, enter the same key in `/etc/cpcache/cpcache.toml` on the server, where cpcache is installed:
```toml
[recv_packages]
    key = "topsecret"
```

Remember to restart the server after having changed the key:
```bash
systemctl restart cpcache
```

You can check if clyde-client successfully authorizes its POST requests by running `/usr/bin/clyde_client` and inspecting the systemd journal. You should then see a message such as `Successfully written file /var/cache/cpcache/wanted_packages/myhost for host myhost`.

On the server, after having installed clyde-server, start and enable `clyde.timer`:
```bash
systemctl start clyde_server.timer
systemctl enable clyde_server.timer
```

clyde-server will now run each night at around 4 o'clock (± one hour). For each package used by at
least one client, it will fetch the most recent version, unless already present.
