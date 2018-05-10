# clyde

clyde is meant to be used in addition to [cpcache](https://github.com/nroi/cpcache).
Its purpose is to keep the cache warm, i.e., to download packages from the cache such that subsequent
HTTP GET requests of the same package can be fulfilled without having to download the file from a remote mirror.


## Installation

### Server Configuration
Before continuing, please make sure that cpcache is running. Check the output of `systemctl status cpcache` to
verify that cpcache is listening on its standard port `7070` for incoming requests. 

First, we need to create a key which will be used to authorize HTTP POST requests sent by clyde-client to cpcache.
This key is shared between all clients and servers, so it has to be generated only once.
The key should not contain any chars that are not suited to be sent in an HTTP request. You can use openssl to
generate a suitable key for you:
```
$ openssl rand -hex 25
34f449e4050605ccd1720071cafd00489e448c95b25c963f69
```

Add this key to the the config file of cpcache. In `/etc/cpcache/cpcache.toml`, search for `recv_packages`. As explained
by the comments in this section of the file, you'll have to uncomment the two lines and add your key:
```TOML
[recv_packages]
    key = "34f449e4050605ccd1720071cafd00489e448c95b25c963f69"
```

Remember to restart cpcache after having changed its configuration:
```bash
systemctl restart cpcache
```

Then, install  [clyde-server](https://aur.archlinux.org/packages/clyde-server-git/) so that you can start and enable the timer:
```bash
systemctl start clyde_server.timer
systemctl enable clyde_server.timer
```

### Client Configuration

Note that we use `cpcache.local` as the hostname for cpache. Replace it by whatever IP address or hostname can be used to reach
cpcache inside your own LAN. Repeat the following steps on all machines that are used as clients (i.e., all machines that
use cpcache as pacman cache):

1. Adapt your `/etc/pacman.d/mirrorlist` so that clyde-client know how to reach cpcache. clyde-client looks for
   a line with a trailing `!cpcache` comment. If cpcache runs on the same machine as the client, you would add
   the following line at the beginning of your mirrorlist file:
   ```bash
   Server = http://cpcache.local:7070/$repo/os/$arch # !cpcache
   ```
2. Create the directory `/etc/clyde_client` and add the key to a file named `key` inside this directory:
   ```bash
   mkdir /etc/clyde_client
   echo 34f449e4050605ccd1720071cafd00489e448c95b25c963f69 > /etc/clyde_client/key
   ```
3. Next, we can continue to install [clyde-client](https://aur.archlinux.org/packages/clyde-client-git/).
   Check the pacman output after installing clyde-client. You should see a message such as:
   ```
   :: Running post-transaction hooks...
   (1/2) Sending list of installed packages to cpcache
   ```
   clyde-client uses pacman hooks so that after each upgrade or package removal run by pacman, all packages required by the
   client will be sent to cpcache.
   To make sure that cpcache successfully receives this message, check the journal of cpcache (e.g. `systemctl status cpcache`).
   It should log a message such as `Successfully written file /var/cache/cpcache/wanted_packages/arch-vm for host arch-vm`.

Now, the server knows which packages are required by inspecting the files inside the directory
`/var/cache/cpcache/wanted_packages`. clyde-server will run each night at around 4 o'clock (± one hour).
For each package used by at least one client, it will fetch the most recent version, unless already present.
