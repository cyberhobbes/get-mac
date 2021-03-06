# DEV (xx/xx/2018)

## Added
* Windows default interface detection if `network_request` is enabled (Credit: @cyberhobbes)
* Docker container (Credit: @Komish)

## Changed
* Use proper Python 2-compatible print functions (Credit: @martmists)

## Removed
* Support for Python 2.5. It is not feasible to test, and potentially
breaks some useful language features, such as `__future__`
* PORT and DEBUG from package imports (`__init__.py`), since changing
these would have no actual effect on execution

## Dev
* Added contribution guide
* Added example videos demonstrating usage (Credit: @fortunate-man)


# 0.5.0 (09/24/2018)
## Added
* Full support for Windows Subsystem for Linux (WSL). This is working for
all features, including default interface selection! The only edge case
is lookup of remote host IP addresses that are actually local interfaces
won't resolve to a MAC (which should be ff-ff-ff-ff-ff-ff).
## Changed
* Require `argparse` if Python version is 2.6 or older
## Dev
* Updated tox tests: added Jython and IronPython, removed 2.6


# 0.4.0 (09/21/2018)
## Added
* New methods for remote host MACs
    * Windows: `arp`
    * POSIX: `arpreq` package
* New methods for interface MACs
    * Windows: `wmic nic`
* DEBUG levels: DEBUG value is now an integer, and increasing it will
increase the amount and verbosity of output. On the CLI, it can be
configured by increasing the amount of characters for the debug argument,
e.g. '-dd' for DEBUG level 2.
* Jython support (Note: on Windows Jython currently only works with interfaces)
* IronPython support

## Changed
* **Significant** performance improvement for remote hosts. Previously,
the average for `get_mac_address(ip='10.0.0.100')` was 1.71 seconds.
Now, the average is `12.7 miliseconds`, with the special case of a unpopulated
arp table being only slightly higher. This was brought about by changes in
how the arp table is populated. The original method was to use the
host's `ping` command to send an ICMP packet to the host. This took time,
which heavily delayed the ability to actually get an address. The solution
is to instead simply send a empty UDP packet to a high port. The port
this packet is sent to can be configured using the module variable `getmac.PORT`.
* "Fixed" resolution of localhost/127.0.0.1 by hardcoding the response.
This should resolve a lot of problematic edge cases. I'm ok with this
for now since I don't know of a case when it isn't all zeroes.
* Greatly increased the reliability of getting host and interface MACs on Windows
* Improved debugging output
* Tightened up the size of `getmac.py`
* Various minor stability and performance improvements
* Add LICENSE to PyPI package

## Removed
* Support for Python 3.2 and 3.3. The total downloads from PyPI with
those versions in August was ~53k and ~407K, respectfully. The majority
of those are likely from automated testing (e.g. TravisCI) and not
actual users. Therefore, I've decided to drop support to simplify
development, especially since before 3.4 the 3.x series was still
very much a  "work in progress".

## Dev
* Added automated tests for Windows using Appveyor
* Tox runner for tests
* Added github.io page
* Improved TravisCI testing


# 0.3.0 (08/30/2018)
## Added
* Attempt to use Python modules if they're installed. This is useful
for larger projects that already have them installed as dependencies,
as they provide a more reliable means of getting information.
    * `psutil`: Interface MACs on all platforms
    * `scapy`: Interface MACs and Remote MACs on all platforms
    * `netifaces`: Interface MACs on Non-Windows platforms
* New methods for remote MACs
    * POSIX: `ip neighbor show`, Abuse of `uuid._arp_getnode()`
* New methods for Interface MACs
    * POSIX: `lanscan -ai` (HP-UX)

## Changed
* Certain critical failures that should never happen will now warn
instead of failing silently.
* Added a sanity check to the `ip6` argument (IPv6 addresses)
* Improved performance in some areas
* Improved debugging output

## Fixed
* Major Bugfix: search of `proc/net/arp` would return shorter addresses in the
same subnet if they came earlier in the sequence. Example: a search for
`192.168.16.2` on Linux would instead return the MAC address of
`192.168.16.254` with no errors or warning whatsoever.
* Significantly improved default interface detection. Default
interfaces are now properly detected on Linux and most other
POSIX platforms with `ip` or `route` commands available, or the
`netifaces` Python module.

## Dev
* Makefile
* Vagrantfile to spin up testing VMs for various platforms using [Vagrant](https://www.vagrantup.com/docs/)
* Added more samples of command output on platforms (Ubuntu 18.04 LTS)


# 0.2.4 (08/26/2018)
## Fixed
* Fixed identification of remote host on OSX
* Resolved hangs and noticable lag that occured when "network_request"
was True (the default)


# 0.2.3 (08/07/2018)
## Fixed
* Remote host for Python 3 on Windows


# 0.2.2
## Added
* Short versions of CLI arguments (e.g. "-i" for "--interface")

## Changed
* Improved usage of "ping" across platforms and IP versions
* Various minor tweaks for performance
* Improved Windows detection

## Fixed
* Use of ping command with hostname

## Dev:
* Improvements to internal code


# 0.2.1
Nothing changed. PyPI just won't let me push changes without a new version.


# 0.2.0 (04/15/2018)
## Added
* Checks for default interface on Linux systems
* New methods of hunting for addresses on Windows, Mac OS X, and Linux

## Changed
* CLI will output nothing if it failed, instead of "None"
* CLI will return with 1 on failure, 0 on success
* No CLI arguments now implies the default host network interface
* Added an argumnent for debugging: `--debug`
* Removed `-d` option from `--no-network-requests`

## Fixed
* Interfaces on Windows and Linux (including Bash for Windows)
* Many bugs

## Removed
* Support for Python 2.6 on the CLI

## Dev
* Overhaul of internals


# 0.1.0 (04/15/2018):
## Added
* Addition of a terminal command: `get-mac`
* Ability to run as a module from the command line: `python -m getmac`

## Changed
* `arp_request` argument was renamed to `network_request`
* Updated docstring
* Slight reduction in the size of getmac.py

## Dev
* Overhauled the README
* Moved tests into their own folder
* Added Python 3.7 to list of supported snakes


# 0.0.4 (11/12/2017):
* Python 2.6 compatibility


# 0.0.3 (11/11/2017):
* Fixed some addresses returning without colons
* Added more rigorous checks on addresses before returning them


# 0.0.2 (11/11/2017):
* Remove print statements and other debugging output


# 0.0.1 (10/23/2017):
* Initial pre-alpha
