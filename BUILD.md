Introduction:
-------------
This tutorial aims to compile DPDK for pfSense.
 
Prerequisites:
--------------
To compile anything for pfSense, you need:
 
- A FreeBSD machine with a kernel version equal to or higher than pfSense.
- Enough disk space (32GB recommended), RAM (8GB recommended), and as much CPU power as possible, since this machine will perform cross-compilation.
- Install required packages: git, wget, poudriere, vim, portmaster, portsnap, and screen by running:
  pkg update && pkg install git wget poudriere vim portmaster portsnap screen
- Become root on FreeBSD.
 
You can check the pfSense kernel version at this link: https://github.com/pfsense/FreeBSD-src
Installing pfSense sources:
---------------------------
1) Getting pfSense sources.
 
Go to the GitHub page for pfSense sources and check the list of branches. For example, for pfSense 2.7.2, select RELENG_2_7_2.
 
If the /usr/src directory isn't empty, clear it: cd /usr && rm -R /usr/src/
 
Download the sources:
git clone --branch RELENG_2_7_2 https://github.com/pfsense/FreeBSD-src /usr/src
 
2) Compiling pfSense programs:
 
You must compile all pfSense binaries to create a build environment for your programs or kernel modules.
 
As an indication, compilation takes around 45 minutes on an Intel i9-9900K (8 cores).
 
To compile:
cd /usr/src/
make buildworld -j numberOfCpuCores
 
Creating the compilation environment:
-------------------------------------
To compile binaries, create a build environment in a FreeBSD Jail using Poudriere.
 
1) Configuring poudriere:
 
Create poudriere configuration file:
cp /usr/local/etc/poudriere.conf.sample /usr/local/etc/poudriere.conf
 
Modify it as follows:
 
- If not using ZFS, set NO_ZFS=yes
- If using ZFS, get pool names with zpool list, then edit the configuration file with:
  ZPOOL=yourpoolname
  ZROOTFS=/poudriere
- Set FREEBSD_HOST=ftp://ftp.freebsd.org
 
2) Creating the environment:
 
Choose a name for your environment, for example pf272amd64 (no periods allowed).
 
Since you're compiling binaries for a kernel that's not the latest version (pfSense 2.7.2) and no longer officially supported, create a configuration file:
vim /usr/local/etc/poudriere.d/environmentname-make.conf
 
And add this line:
ALLOW_UNSUPPORTED_SYSTEM=yes
 
Create a ports tree:
poudriere ports -c -p HEAD
 
Remove unnecessary dependencies:
pkg autoremove
 
Create your jail environment, installing binaries compiled from pfSense sources:
poudriere jail -c -j pf272amd64 -a amd64 -m src=/usr/src -p HEAD
 
Retrieving ports sources:
-------------------------
To retrieve the ports list:
portsnap fetch update
 
Verify the list by checking /usr/ports and browsing available ports.
 
Alternatively, clone ports with git:
git clone https://github.com/freebsd/freebsd-ports /usr/ports
 
Compiling the desired port:
---------------------------
1) Creating a list of ports to compile:
 
Create a file containing the port path from /usr/ports to the root of the port directory you want. For example:
echo net/dpdk > pkglist.txt
 
2) Compiling the port:
 
If connected via SSH, open a screen session:
screen
 
You can detach from this session at any time using CTRL-a d, and later reconnect with:
screen -x
 
Start compilation:
poudriere bulk -f pkglist.txt -j pf272amd64 -p HEAD
 
Monitor progress with CTRL - t
 
3) Retrieving compiled packages:
 
Compiled packages are located in:
/usr/local/poudriere/data/packages/pf272amd64-HEAD/All/
 
Installing packages:
--------------------
To install these packages, retrieve all packages from the same compilation (due to dependencies), then install the desired package:
pkg add nameofpackagetoinstall.pkg
