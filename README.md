
[![Latest version on PyPI](https://badge.fury.io/py/get-mac.svg)](https://pypi.org/project/get-mac/)
[![Travis CI build status](https://travis-ci.org/GhostofGoes/get-mac.svg?branch=master)](https://travis-ci.org/GhostofGoes/get-mac)
[![Appveyor build status](https://ci.appveyor.com/api/projects/status/4o9mx4d35adrbssq/branch/master?svg=true)](https://ci.appveyor.com/project/GhostofGoes/get-mac/branch/master)

Get the MAC address of remote hosts or network interfaces using Python.

It provides a platform-independant interface to get the MAC addresses of:

* System network interfaces (by interface name)
* Remote hosts on the local network (by IPv4/IPv6 address or hostname)

It provides one function: `get_mac_address()`

[![asciicast](https://asciinema.org/a/rk6dUACUcZY18taCuIBE5Ssus.png)](https://asciinema.org/a/rk6dUACUcZY18taCuIBE5Ssus)

[![asciicast](https://asciinema.org/a/n3insrxfyECch6wxtJEl3LHfv.png)](https://asciinema.org/a/n3insrxfyECch6wxtJEl3LHfv)

## Features
* Pure-Python
* Supports Python 2.6+, 3.4+, pypy, and pypy3
* No dependencies
* Small size
* Can be used as an independant .py file
* Simple terminal tool (when installed as a package)

# Usage

## Python examples
```python
from getmac import get_mac_address
eth_mac = get_mac_address(interface="eth0")
win_mac = get_mac_address(interface="Ethernet 3")
ip_mac = get_mac_address(ip="192.168.0.1")
ip6_mac = get_mac_address(ip6="::1")
host_mac = get_mac_address(hostname="localhost")
updated_mac = get_mac_address(ip="10.0.0.1", network_request=True)

# Enabling debugging
from getmac import getmac
getmac.DEBUG = 2  # DEBUG level 2
print(getmac.get_mac_address(interface="Ethernet 3"))

# Changing the port used for updating ARP table (UDP packet)
from getmac import getmac
getmac.PORT = 44444  # Default: 55555
print(get_mac_address(ip="192.168.0.1", network_request=True))
```

## Terminal examples
```bash
get-mac --help
get-mac --version

# No arguments will return MAC of the default interface.
get-mac
python -m getmac

# Interface names, IPv4/IPv6 addresses, or Hostnames can be specified
get-mac --interface ens33
get-mac --ip 192.168.0.1
get-mac --ip6 ::1
get-mac --hostname home.router

# Running as a Python module with shorthands for the arguments
python -m getmac -i 'Ethernet 4'
python -m getmac -4 192.168.0.1
python -m getmac -6 ::1
python -m getmac -n home.router

# Getting the MAC address of a remote host obviously requires
# the ARP table to be populated. By default, get-mac will do
# this for you by sending a small UDP packet to a high port (55555)
# If you don't want this to happen, you can disable it.
# This is useful if you're 100% certain the ARP table will be
# populated already, or in red team/forensic scenarios.
get-mac --no-network-request -4 192.168.0.1
python -m getmac --no-network-request -n home.router

# Debug levels can be specified with '-d'
get-mac --debug
python -m getmac -d -i enp11s4
python -m getmac -dd -n home.router
```

Note: the terminal interface will not work on Python 2.6 and older (Sorry RHEL 6 users!).

## get_mac_address()
* `interface`: Name of a network interface on the system.
* `ip`: IPv4 address of a remote host.
* `ip6`: IPv6 address of a remote host.
* `hostname`: Hostname of a remote host.
* `network_request`: If an network request should be made to update
and populate the ARP/NDP table of remote hosts used to lookup MACs
in most circumstances. Disable this if you want to just use what's
already in the table, or if you have requirements to prevent network
traffic. The network request is a empty UDP packet sent to a high
port, 55555 by default. This can be changed by setting `getmac.PORT`
to the desired integer value. Additionally, on Windows, this will
send a UDP packet to 1.1.1.1:53 to attempt to determine the default interface.

## Notes
* If none of the arguments are selected, the default
network interface for the system will be used.
* "Remote hosts" refer to hosts in your local layer 2 network, also
commonly referred to as a "broadcast domain", "LAN", or "VLAN". As far
as I know, there is not a reliable method to get a MAC address for a
remote host external to the LAN. If you know any methods otherwise, please
open a GitHub issue or shoot me an email, I'd love to be wrong about this.
* The first four arguments are mutually exclusive. `network_request`
does not have any functionality when the `interface` argument is
specified, and can be safely set if using in a script.
* The physical transport is assumed to be Ethernet (802.3). Others, such as
Wi-Fi (802.11), are currently not tested or considored. I plan to
address this in the future, and am definitely open to pull requests
or issues related to this, including error reports.
* Exceptions will be handled silently and returned as a None.
    If you run into problems, you can set DEBUG to true and get more
    information about what's happening. If you're still having issues,
    please create an issue on GitHub and include the output with DEBUG enabled.
* Messages are output using the `warnings` module, and `print()` if
`getmac.DEBUG` enabled (any value greater than 0).
If you are using logging, they can be captured using logging.captureWarnings().
Otherwise, they can be suppressed using warnings.filterwarnings("ignore").
https://docs.python.org/3/library/warnings.html

## Docker Examples
Build docker image (from repository root directory)

```bash
docker build . -t get-mac
```

Run get-mac container and provide flags

```bash
docker run -it get-mac:latest --help
docker run -it get-mac:latest --version
docker run -it get-mac:latest -n localhost
```

# Commands and techniques by platform
* Windows
    * Commands: `getmac`, `ipconfig`
    * Libraries: `uuid`, `ctypes`
    * Third-party Packages: `netifaces`, `psutil`, `scapy`
* Linux/Unix
    * Commands: `arp`, `ip`, `ifconfig`, `netstat`, `ip link`
    * Libraries: `uuid`, `fcntl`
    * Third-party Packages:  `netifaces`, `psutil`, `scapy`, `arping`
    * Default interfaces: `route`, `ip route list`
    * Files: `/sys/class/net/X/address`, `/proc/net/arp`
* Mac OSX (Darwin)
    * `networksetup`
    * Same commands as Linux
* WSL: Windows commands are used for remote hosts,
and Unix commands are used for interfaces

# Platforms currently supported
All or almost all features should work on "supported" platforms (OSes).
* Windows
    * Desktop: 7, 8, 8.1, 10
    * Server: TBD
    * (Partially supported, untested): 2000, XP, Vista
* Linux distros
    * CentOS/RHEL 6+
    * Ubuntu 16+ (14 and older should work as well)
    * Fedora
* MacOSX (Darwin)
    * The latest two versions probably (TBD)
* Windows Subsystem for Linux (WSL)
* Docker

# Python versions
All sub-versions are the latest available on your platform (with the exception of 2.6).
* 2.6.6 (CentOS 6/RHEL 6 version)
* 2.7
* 3.4
* 3.5
* 3.6
* 3.7

# Caveats & Known issues
Please report any problems by opening a issue on GitHub!

## Caveats
* Depending on the platform, there could be a performance detriment,
due to heavy usage of regular expressions.
* Platform test coverage is imperfect. If you're having issues,
then you might be using a platform I haven't been able to test.
Keep calm, open a GitHub issue, and I'd be more than happy to help.
* Older Python versions (2.5/3.3 and older) are not officially supported.
If you're running these, all is not lost! Simply copy/paste `getmac.py`
into your codebase and make the neccesary edits to be compatible with
your version and distribution of Python.

## Known Issues
* Hostnames for IPv6 devices are not yet supported.
* Windows: the "default" of selecting the default route interface only
works effectively if `network_request` is enabled. Otherwise,
`Ethernet` as the default.
* There is are currently no automated tests for Python 2.6, which means
there is a much higher potential for regressions. Open an issue if you
encounter any.

# Contributing
Contributers are more than welcome!
See the [contribution guide](CONTRIBUTING.md) to get started,
and checkout the [todo list](TODO.md) for a full list of tasks and bugs.

Before submitting a PR, please make sure you've completed the
[pull request checklist](CONTRIBUTING.md#Code_requirements)!

The [Python Discord server](https://discord.gg/python) is a good place
to ask questions or discuss the project (Handle: @KnownError).

## Contributors
* Christopher Goes (@ghostofgoes) - Author and maintainer
* Calvin Tran (@cyberhobbes) - Windows interface detection improvements
* Jose Gonzalez (@Komish) - Docker container and Docker testing
* @fortunate-man - Awesome usage videos
* @martmists - legacy Python compatibility improvements

# Sources
Many of the methods used to acquire an address and the core logic framework
are attributed to the CPython project's UUID implementation.
* https://github.com/python/cpython/blob/master/Lib/uuid.py
* https://github.com/python/cpython/blob/2.7/Lib/uuid.py
* [_unix_fcntl_by_interface](https://stackoverflow.com/a/4789267/2214380)
* [_windows_get_remote_mac_ctypes](goo.gl/ymhZ9p)
* [String joining](https://stackoverflow.com/a/3258612/2214380)

# License
MIT. Feel free to copy, modify, and use to your heart's content. Enjoy :)
