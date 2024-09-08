---
title: RTL-SDR V4 and a Pi B
description: Testing an RTL-SDR BLOG V4 with a Rasberry PI Model B Rev2
date: 2024-08-29 04:00:01 +0000
categories: [projects, rtlsdr]
tags: [rtl-sdr, raspberrypi, sdr]
media_subpath: /assets/img/2024/08/
image:
    path: 29-b-ssh-disconnect.png
---

> TL;DR This below didn't go well, the Pi kept rebooting when the SDR was connected. I assume hardware related.
> Next is [an attempt with a newer Pi 3]({% post_url 2024-08-31-rtlsdr-pi-3bp %}).
{: .prompt-danger }

> Approach fyi & in case I refer back to steps here.
{: .prompt-tip }

Having [tested the RTL-SDR Blog V2 on my laptop]({% post_url 2024-08-27-rtl-sdr-blog-v4 %}) I next up is to see if it will run on my Raspberry PI, a [2012 Pi Model B Rev2]({% post_url 2024-08-28-raspberry-pi-2011-modelb %})

On this ocassion I'll be trying the Model B Rev2. 

## RTL-SDR Driver install

Per [the V4 Users Guide](https://www.rtl-sdr.com/V4/), the approach to getting a driver for the various systems varies. For Debian GNU/Linux, the driver is grabbed [from their github](https://github.com/rtlsdrblog/rtl-sdr-blog) and compiled.


### NB - they have two methods
1. building the source and using make install
1. building via dpkg-buildpackage

I'm going to try the first one...

## Source build
### Cleaning old drivers from your system
1. Initial steps are purging older drivers:
```shell
sudo apt purge ^librtlsdr
```
but I didn't have any of those installed ([output](#appendix-a-output---removing-old-drivers)).

1. They to remove previous user-built drivers if any:
```shell
sudo rm -rvf /usr/lib/librtlsdr* /usr/include/rtl-sdr* /usr/local/lib/librtlsdr* /usr/local/include/rtl-sdr* /usr/local/include/rtl_* /usr/local/bin/rtl_*
```
but again i didn't have any of that either ([output](#appendix-b-output---purging-old-user-built-drivers)).

### Preparing for Source Build
1. Getting Driver build pre-requisites.
	- Before grabbing their driver code and building, you need to make sure you have a few pieces ready. 
	- In addition to the things specified here, you will also see those each may have further dependancies which will also be installed.
```shell
sudo apt-get install libusb-1.0-0-dev git cmake pkg-config 
```
	I initially  got an error there, but after running a quick package-manager update via `sudo apt-get update` and then a retry it worked fine. 
([output](#appendix-c-output---installing-dependancies))


1. Grabbing the code
Grabbing their code is a simple `git` command, and it will create a folder called by the name of the repository (`rtl-sdr-blog`). Or you can use `git clone <repo> <target-directory-name>` I think.
```shell
git clone https://github.com/rtlsdrblog/rtl-sdr-blog
```
([output](#appendix-d-output---git-clone))


1. Checking the system, build tools and config are next . I hadn't used `cmake` before, and read around to find it is a successor of `./configure` for generating a `Makefile` after performing some platform and environment checks. More [on their wiki](https://gitlab.kitware.com/cmake/community/-/wikis/FAQ#what-is-cmake). Those few prep commands involved in this step comprise:
```shell
cd rtl-sdr-blog/
mkdir build
cd build
cmake ../ -DINSTALL_UDEV_RULES=ON
```
and again, [output](#appendix-e-output---build-prep) for reference.


### Building
At this stage, we're ready to build the driver. This command is as easy at it gets, just type `make` :D
```shell
make
```
This takes about 6minutes on the Model B Pi. [output](#appendix-f-output---build).

### Install the built driver
Down to the final hurdles. 
1. Install will copy the built libraries and make them available for runtime use
```shell
sudo make install
```
[output](#appendix-g-output---make-install).

1. Copy the driver rules for udev, becuase _"...udev allows for rules that specify what name is given to a device, regardless of which port it is plugged into.... This consistent naming of devices guarantees that scripts dependent on a specific device's existence will not be broken."_ [ref](https://wiki.debian.org/udev#:~:text=udev%20allows%20for%20rules%20that,%2Fdev%2Firiver%20is%20possible.)
```console
pi@rasppi:~/rtl-sdr-blog/build $ sudo cp ../rtl-sdr.rules /etc/udev/rules.d/ 
```


1. Configure the dynamic linker as _"...ldconfig creates the necessary links and cache to the most recent shared libraries..."_ [ref](https://man7.org/linux/man-pages/man8/ldconfig.8.html)
```console
pi@rasppi:~/rtl-sdr-blog/build $ sudo ldconfig
```

1. And blacklist some DVB-T TV Drivers so they aren't inadvertently used instead I guess.
```console
pi@rasppi:~/rtl-sdr-blog/build $ echo 'blacklist dvb_usb_rtl28xxu' | sudo tee --append /etc/modprobe.d/blacklist-dvb_usb_rtl28xxu.conf
blacklist dvb_usb_rtl28xxu
```

## Reboot
This is all wrapped up in `sudo shutdown -r now` :
```console
pi@rasppi:~/rtl-sdr-blog/build $ sudo shutdown -r now
Connection to rasppi closed by remote host.
Connection to rasppi closed.
```

## Connect the SDR
I wasn't sure how the PI would behave when the SDR was connected, so I ran `dmesg -w` to tail the system messages.

### Trouble
> Unfortunately, the next message i saw was a couple of minutes later - i got disconnected.
{: .prompt-warning }
```console
client_loop: send disconnect: Broken pipe
```

I unplugged the RTL SDR device, and was able to reconnect to the Pi, but by that time it seemed like what had happened was the system had reset/restarted.

Sure enough when i checked the "update" it reported only 3 minutes !
```console
pi@rasppi:~ $ uptime
20:34:19 up 3 min,  2 users,  load average: 0.34, 0.57, 0.27
```


### Troubleshooting
Unfortunately checking various logs doesn't offer any insights...
```shell
tail /var/log/syslog
tail /var/log/daemon.log
tail /var/log/kern.log
```

...everything seems fine update until 28minutes after the hour, and then nothing until 31minutes after, which is startup related messages :(

Various[<sup>1</sup>](https://forums.raspberrypi.com/viewtopic.php?t=258722) threads[<sup>2</sup>](https://raspberrypi.stackexchange.com/questions/114110/raspberry-pi-random-reboots-raspi-3b) online[<sup>3</sup>](https://stackoverflow.com/questions/38209074/rasperry-pi-reboots-when-running-program) point to power issues as the most likely culprit.

The second link there suggests how [`vcgencmd`](https://www.raspberrypi.com/documentation/computers/os.html#vcgencmd) may be used to check throttling, temperature and voltage.

I wrote a little script that runs each of `vcgencmd` on a loop with a `sleep` for 1 second between each round, hoping it might offer some clues about the restarts:
```bash
#!/bin/bash
function checkem ()
{
	echo `date` >> vc-throttle.txt
	vcgencmd get_throttled >> vc-throttle.txt

	echo `date` >>  vc-temp.txt
	vcgencmd measure_temp >> vc-temp.txt

	echo `date` >> vc-volt.txt
	vcgencmd measure_volts >> vc-volt.txt
}

while [[ true ]] ; do
	checkem
	sleep 2
done
```

I also had a look at reducing power use, e.g. [this article from 2021](https://blues.com/blog/tips-tricks-optimizing-raspberry-pi-power/) about power optimisation. 
- Powering off HDMI and maybe the L.E.D.s seem like good options. 
- Unsure about down-clocking CPU. 
- USB is needed for the SDR
- this Pi doesnt' have Wifi/Bluetooth.

To turn off HDMI to see if I could get a little extra juice, this worked on the OS Version (Raspbian GNU/Linux 10 (buster))
```console
pi@rasppi:~ $ tvservice -o
Powering off HDMI
```
    
While I'm typing this, it has remained on .....


## Testing the driver and device
There are various command-line options available to test the SDR, without needing to install anything further.

```console
pi@rasppi:~ $ rtl_
rtl_adsb    rtl_biast   rtl_eeprom  rtl_fm      rtl_power   rtl_sdr     rtl_tcp     rtl_test    
```

### rtl_test
`rtl_test`, _"a benchmark tool for RTL2832 based DVB-T receivers"_, seems like a nice simple option so I gave that a go.
```console
pi@rasppi:~ $ rtl_test -p
Found 1 device(s):
  0:  RTLSDRBlog, Blog V4, SN: 00000001

Using device 0: Generic RTL2832U OEM
Found Rafael Micro R828D tuner
RTL-SDR Blog V4 Detected
Supported gain values (29): 0.0 0.9 1.4 2.7 3.7 7.7 8.7 12.5 14.4 15.7 16.6 19.7 20.7 22.9 25.4 28.0 29.7 32.8 33.8 36.4 37.2 38.6 40.2 42.1 43.4 43.9 44.5 48.0 49.6 
Sampling at 2048000 S/s.
Reporting PPM error measurement every 10 seconds...
Press ^C after a few minutes.
Reading samples in async mode...
lost at least 20 bytes
```
After the report of "lost" bytest, nothing else arrived at the console.

Background for that tool is [at this link](https://discussions.flightaware.com/t/understanding-rtl-test-p-results/16660). 
> That discussion mentions [`dump1090`](https://github.com/antirez/dump1090), a utility for handling `Mode S` (ADS-B) messages transmitted from aircraft on `1090 MHz` with heading, speed, altitude etc for colission avoidance. Fun for future!
{: .prompt-info }

> More trouble :( After a while, I tried to kill that `rtl_test` command, but again the device had reset. 
{: .prompt-warning }


Checking the throttle, temp and volt outputs (had been running in the background) showed no obvious culprit...

I'm now thinking of a donated Pi3 I have here too, time to switch devices perhaps. To be continued...


## Appendices
### Appendix A: Output - Removing old drivers
```console
pi@rasppi:~ $ sudo apt purge ^librtlsdr
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Note, selecting 'librtlsdr-dev' for regex '^librtlsdr'
Note, selecting 'librtlsdr0' for regex '^librtlsdr'
Note, selecting 'librtlsdr0-dbgsym' for regex '^librtlsdr'
Package 'librtlsdr-dev' is not installed, so not removed
Package 'librtlsdr0' is not installed, so not removed
Package 'librtlsdr0-dbgsym' is not installed, so not removed
0 upgraded, 0 newly installed, 0 to remove and 156 not upgraded.
```

[back](#cleaning-old-drivers-from-your-system)

### Appendix B: Output - Purging old user-built drivers
I did double check via `ls` to be sure.
```console
pi@rasppi:~ $ sudo ls -al /usr/lib/librtlsdr* /usr/include/rtl-sdr* /usr/local/lib/librtlsdr* /usr/local/include/rtl-sdr* /usr/local/include/rtl_* /usr/local/bin/rtl_*
ls: cannot access '/usr/lib/librtlsdr*': No such file or directory
ls: cannot access '/usr/include/rtl-sdr*': No such file or directory
ls: cannot access '/usr/local/lib/librtlsdr*': No such file or directory
ls: cannot access '/usr/local/include/rtl-sdr*': No such file or directory
ls: cannot access '/usr/local/include/rtl_*': No such file or directory
ls: cannot access '/usr/local/bin/rtl_*': No such file or directory
```

[back](#cleaning-old-drivers-from-your-system)


### Appendix C: Output - installing dependancies
```console
pi@rasppi:~ $ sudo apt-get install libusb-1.0-0-dev git cmake pkg-config 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
pkg-config is already the newest version (0.29-6).
The following additional packages will be installed:
  cmake-data git-man libarchive13 liberror-perl libjsoncpp1 librhash0 libusb-1.0-doc libuv1
Suggested packages:
  cmake-doc ninja-build git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-cvs git-mediawiki git-svn lrzip
The following NEW packages will be installed:
  cmake cmake-data git git-man libarchive13 liberror-perl libjsoncpp1 librhash0 libusb-1.0-0-dev libusb-1.0-doc libuv1
0 upgraded, 11 newly installed, 0 to remove and 156 not upgraded.
Need to get 10.8 MB of archives.
After this operation, 57.4 MB of additional disk space will be used.
Do you want to continue? [Y/n] yes
```

[back](#preparing-for-source-build)

### Appendix D: Output - git clone
```console
pi@rasppi:~ $ git clone https://github.com/rtlsdrblog/rtl-sdr-blog 
Cloning into 'rtl-sdr-blog'...
remote: Enumerating objects: 2314, done.
remote: Counting objects: 100% (875/875), done.
remote: Compressing objects: 100% (135/135), done.
remote: Total 2314 (delta 792), reused 740 (delta 740), pack-reused 1439 (from 1)
Receiving objects: 100% (2314/2314), 737.97 KiB | 671.00 KiB/s, done.
Resolving deltas: 100% (1651/1651), done.
```

[back](#preparing-for-source-build)

### Appendix E: Output - build prep
```console
pi@rasppi:~ $ cd rtl-sdr-blog 
pi@rasppi:~/rtl-sdr-blog $ mkdir build 
pi@rasppi:~/rtl-sdr-blog $ cd build 
pi@rasppi:~/rtl-sdr-blog/build $ cmake ../ -DINSTALL_UDEV_RULES=ON 
-- The C compiler identification is GNU 8.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Build type not specified: defaulting to release.
-- Extracting version information from git describe...
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Failed
-- Looking for pthread_create in pthreads
-- Looking for pthread_create in pthreads - not found
-- Looking for pthread_create in pthread
-- Looking for pthread_create in pthread - found
-- Found Threads: TRUE  
-- Found PkgConfig: /usr/bin/pkg-config (found version "0.29") 
-- Checking for module 'libusb-1.0'
--   Found libusb-1.0, version 1.0.22
-- Building with kernel driver detaching disabled, use -DDETACH_KERNEL_DRIVER=ON to enable
-- Building with usbfs zero-copy support disabled, use -DENABLE_ZEROCOPY=ON to enable
-- Performing Test COMPILER_HAS_HIDDEN_VISIBILITY
-- Performing Test COMPILER_HAS_HIDDEN_VISIBILITY - Success
-- Performing Test COMPILER_HAS_HIDDEN_INLINE_VISIBILITY
-- Performing Test COMPILER_HAS_HIDDEN_INLINE_VISIBILITY - Failed
-- Performing Test COMPILER_HAS_DEPRECATED_ATTR
-- Performing Test COMPILER_HAS_DEPRECATED_ATTR - Success
-- Building for version: 0.6.0-128-g240b / 0.6git
-- Using install prefix: /usr/local
-- Configuring done
-- Generating done
-- Build files have been written to: /home/pi/rtl-sdr-blog/build
```

[back](#preparing-for-source-build)


### Appendix F: Output - build
```console
pi@rasppi:~/rtl-sdr-blog/build $ $ make
Scanning dependencies of target convenience_static
[  3%] Building C object src/CMakeFiles/convenience_static.dir/convenience/convenience.c.o
[  6%] Linking C static library libconvenience_static.a
[  6%] Built target convenience_static
Scanning dependencies of target rtlsdr
[  9%] Building C object src/CMakeFiles/rtlsdr.dir/librtlsdr.c.o
[ 12%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_e4k.c.o
[ 15%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_fc0012.c.o
[ 18%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_fc0013.c.o
[ 21%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_fc2580.c.o
[ 25%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_r82xx.c.o
[ 28%] Linking C shared library librtlsdr.so
[ 28%] Built target rtlsdr
Scanning dependencies of target rtl_test
[ 31%] Building C object src/CMakeFiles/rtl_test.dir/rtl_test.c.o
[ 34%] Linking C executable rtl_test
[ 34%] Built target rtl_test
Scanning dependencies of target rtl_biast
[ 37%] Building C object src/CMakeFiles/rtl_biast.dir/rtl_biast.c.o
[ 40%] Linking C executable rtl_biast
[ 40%] Built target rtl_biast
Scanning dependencies of target rtl_tcp
[ 43%] Building C object src/CMakeFiles/rtl_tcp.dir/rtl_tcp.c.o
[ 46%] Linking C executable rtl_tcp
[ 46%] Built target rtl_tcp
Scanning dependencies of target rtl_sdr
[ 50%] Building C object src/CMakeFiles/rtl_sdr.dir/rtl_sdr.c.o
[ 53%] Linking C executable rtl_sdr
[ 53%] Built target rtl_sdr
Scanning dependencies of target rtl_power
[ 56%] Building C object src/CMakeFiles/rtl_power.dir/rtl_power.c.o
[ 59%] Linking C executable rtl_power
[ 59%] Built target rtl_power
Scanning dependencies of target rtlsdr_static
[ 62%] Building C object src/CMakeFiles/rtlsdr_static.dir/librtlsdr.c.o
[ 65%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_e4k.c.o
[ 68%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_fc0012.c.o
[ 71%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_fc0013.c.o
[ 75%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_fc2580.c.o
[ 78%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_r82xx.c.o
[ 81%] Linking C static library librtlsdr.a
[ 81%] Built target rtlsdr_static
Scanning dependencies of target rtl_eeprom
[ 84%] Building C object src/CMakeFiles/rtl_eeprom.dir/rtl_eeprom.c.o
[ 87%] Linking C executable rtl_eeprom
[ 87%] Built target rtl_eeprom
Scanning dependencies of target rtl_fm
[ 90%] Building C object src/CMakeFiles/rtl_fm.dir/rtl_fm.c.o
[ 93%] Linking C executable rtl_fm
[ 93%] Built target rtl_fm
Scanning dependencies of target rtl_adsb
[ 96%] Building C object src/CMakeFiles/rtl_adsb.dir/rtl_adsb.c.o
[100%] Linking C executable rtl_adsb
[100%] Built target rtl_adsb
```

[back](#building)


### Appendix G: Output - make install
```console
pi@rasppi:~/rtl-sdr-blog/build $ sudo make install
[  6%] Built target convenience_static
[ 28%] Built target rtlsdr
[ 34%] Built target rtl_test
[ 40%] Built target rtl_biast
[ 46%] Built target rtl_tcp
[ 53%] Built target rtl_sdr
[ 59%] Built target rtl_power
[ 81%] Built target rtlsdr_static
[ 87%] Built target rtl_eeprom
[ 93%] Built target rtl_fm
[100%] Built target rtl_adsb
Install the project...
-- Install configuration: "Release"
-- Installing: /etc/udev/rules.d/rtl-sdr.rules
-- Installing: /usr/local/include/rtl-sdr.h
-- Installing: /usr/local/include/rtl-sdr_export.h
-- Installing: /usr/local/lib/pkgconfig/librtlsdr.pc
-- Installing: /usr/local/lib/cmake/rtlsdr/rtlsdrTargets.cmake
-- Installing: /usr/local/lib/cmake/rtlsdr/rtlsdrTargets-release.cmake
-- Installing: /usr/local/lib/cmake/rtlsdr/rtlsdrConfig.cmake
-- Installing: /usr/local/lib/cmake/rtlsdr/rtlsdrConfigVersion.cmake
-- Installing: /usr/local/lib/librtlsdr.so.0.6git
-- Installing: /usr/local/lib/librtlsdr.so.0
-- Installing: /usr/local/lib/librtlsdr.so
-- Installing: /usr/local/lib/librtlsdr.a
-- Installing: /usr/local/bin/rtl_sdr
-- Set runtime path of "/usr/local/bin/rtl_sdr" to ""
-- Installing: /usr/local/bin/rtl_tcp
-- Set runtime path of "/usr/local/bin/rtl_tcp" to ""
-- Installing: /usr/local/bin/rtl_test
-- Set runtime path of "/usr/local/bin/rtl_test" to ""
-- Installing: /usr/local/bin/rtl_fm
-- Set runtime path of "/usr/local/bin/rtl_fm" to ""
-- Installing: /usr/local/bin/rtl_eeprom
-- Set runtime path of "/usr/local/bin/rtl_eeprom" to ""
-- Installing: /usr/local/bin/rtl_adsb
-- Set runtime path of "/usr/local/bin/rtl_adsb" to ""
-- Installing: /usr/local/bin/rtl_power
-- Set runtime path of "/usr/local/bin/rtl_power" to ""
-- Installing: /usr/local/bin/rtl_biast
-- Set runtime path of "/usr/local/bin/rtl_biast" to ""
```

[back](#install-the-built-driver)