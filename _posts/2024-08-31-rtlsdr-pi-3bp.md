---
title: RTL-SDR V4 and a Pi3
description: Testing an RTL-SDR BLOG V4 with a Rasberry PI 3 B+
date: 2024-08-31 00:00:01 +0000
categories: [projects, rtlsdr]
tags: [rtl-sdr, raspberrypi, sdr]
media_subpath: /assets/img/2024/08/
image:
    path: 31-pi3rtl-success.png
    alt: picture of output from rtl_test command on pi3, success!
---

I recently wrote about a failed attempt to [use the RTL-SDL on a Pi Model B]({% post_url 2024-08-29-rtlsdr-pib %}). I have been using the SDR without issue on the laptop with GQRX aslo, so we know the SDR works :D.

I'm going to try again with a [Raspberry PI 3, B+, R1.3]({% post_url 2024-08-28-raspberry-pi-models %})
 and see if that fares any better.


## Raspberry configuration customisations 
Before getting started, I made a few updates to the system config to not bother with the Desktop and other things I won't be using for now. These are optional and I'm sure it's fine to skip, just my preference.


### Raspi-Config

Start with `sudo raspi-config`, I changed these sections:
- 1.S5.B1, Boot: selected Console, text console requiring login (no desktop)
- 2.D4, Composite Video: disabled composite video
- 3, interface options. Disabled each of 
    - I2, RPi connect
    - I3, VNC
    - I4, SPI
    - I5, I2C
    - I6, Serial
    - I7, 1-Wire
    - I8, Remote GPIO


## RTL-SDR Driver Install
For this pi, I followed nearly the exact same RTL-SDR Driver install steps 
[as I did for the Pi Model B]({% post_url 2024-08-29-rtlsdr-pib %}). There may be a bit more commentary there if you want to look. I'm not going to repeat all that, just rip through the steps, capture output and note any differences I go with.

I did get distracted by the fact that the OS I had on the card for this Pi was based on `Debian GNU/Linux "Stretch"`(2017), and since then have been multiple newer versions called `Buster`(2019), `Bullseye`(2021) and now `Bookworm`(2023). I had a few isssues getting the dependancies on the much older `Stretch/9.4` OS and so decided to just upgrade (to `Bookworm`/`12.7`).

Back to Driver installs, this is essentially the same as what we did for the Pi Model B, I will call out any variation and again include output in an appendix.


### Cleaning out any old drivers
*Same* as for the Pi B approach, nothing to do for me there after dealing with the upgrade.


### Getting Driver build pre-requisites
Run the build dependancies install the *same* as before with the Pi B
```shell
sudo apt-get install libusb-1.0-0-dev git cmake pkg-config
``` 

[Output](#appendix-a-build-prereqs-install-output)


### Grabbing the code and preparing it for build 
 *Same* as before with the Pi B
```shell
mkdir ~/devel; cd ~/devel
git clone https://github.com/rtlsdrblog/rtl-sdr-blog 
cd rtl-sdr-blog
mkdir build 
cd build 
cmake ../ -DINSTALL_UDEV_RULES=ON 
```
...well, apart from putting it all in a new `devel` folder.

[Output](#appendix-b-build-config-prep-output)


### Build was the *same*.
*Same* as before with the Pi B, just run `make`. 

[Output](#appendix-c-build-make-output)


### Install of the built driver, rules and dynamic linker 
Were all *same* again as before with the Pi B
```shell
sudo make install
sudo cp ../rtl-sdr.rules /etc/udev/rules.d/ 
sudo ldconfig
```

[Output](#appendix-d-install-output)


### Blacklisting DVB-T/TV Drivers 
Also *same*
```shell
echo 'blacklist dvb_usb_rtl28xxu' | sudo tee --append /etc/modprobe.d/blacklist-dvb_usb_rtl28xxu.conf blacklist dvb_usb_rtl28xxu 
```

[Output](#appendix-e-blacklist-dvb-t)


## Reboot Time!
```shell
 sudo shutdown -r now
 ```

## Quick Test
I'm going to run a test now before doing anything else.
I wanted to open multiple terminals and follow the logs in case of any issues with something like:
```shell
tail -f /var/log/syslog ; tail -f /var/log/daemon.log  ; tail -f /var/log/kern.log
```

Alas in [Debian GNU/Linux 12 they have all moved](https://serverfault.com/questions/1148725/where-is-some-os-logs-in-debian-12).

### Monitoring the Pi
That simplifies things, we can use `journalctl` with a follow option: 
```shell
journalctl -f
```

In a second terminal we can watch the kernel ring buffer output:
```shell
dmesg -w
```

Now it's just a matter of attaching the RTL.

Ok, its' still running, output from above see [Appendix F](#appendix-f-monitor-output-for-device-connect)

There were a few `rtl_*` programs I found previously, so going to have a quick look at those again.


### rtl_test
The first was from reading some discussion on flightaware, thanks to 'tjowen' and others [for this discussion](https://discussions.flightaware.com/t/understanding-rtl-test-p-results/16660).

If you run it with no options, it lets you know this:
> Info: This tool will continuously read from the device, and report if samples get lost. If you observe no further output, everything is fine.

So again I ran `rtl_test -p` (just because I have a baseline/expectation for that), and immediately things are looking at little better. Whereas with [Pi Model B]({% post_url 2024-08-29-rtlsdr-pib %}) we immediately saw "lost bytes" and little else...:
```console
...
Reading samples in async mode...
lost at least 20 bytes
```

...this time i'm seeing a different kind of output:
```console
pi@raspberrypi:~ $ rtl_test -p
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
real sample rate: 2047573 current PPM: -208 cumulative PPM: -208
real sample rate: 2048107 current PPM: 53 cumulative PPM: -77
real sample rate: 2047997 current PPM: -1 cumulative PPM: -52
real sample rate: 2048008 current PPM: 4 cumulative PPM: -38
real sample rate: 2047885 current PPM: -56 cumulative PPM: -41
real sample rate: 2048112 current PPM: 55 cumulative PPM: -25
real sample rate: 2047890 current PPM: -53 cumulative PPM: -29
real sample rate: 2047997 current PPM: -1 cumulative PPM: -26
real sample rate: 2048114 current PPM: 56 cumulative PPM: -17
real sample rate: 2047891 current PPM: -53 cumulative PPM: -20
real sample rate: 2048111 current PPM: 54 cumulative PPM: -14
real sample rate: 2047990 current PPM: -4 cumulative PPM: -13
real sample rate: 2047884 current PPM: -56 cumulative PPM: -16
real sample rate: 2048013 current PPM: 6 cumulative PPM: -15
real sample rate: 2047992 current PPM: -4 cumulative PPM: -14
real sample rate: 2048116 current PPM: 57 cumulative PPM: -9
real sample rate: 2047998 current PPM: -1 cumulative PPM: -9
real sample rate: 2047894 current PPM: -52 cumulative PPM: -11
^CSignal caught, exiting!

User cancel, exiting...
Samples per million lost (minimum): 0
```

How exciting!

### rtl_power
With that popped into the "Success!" bucket, on to another check that caused problems for me previously with the Model B. That was trying to use `rtl_power` courtesy of [this post on Charlie's Labs](https://charleslabs.fr/en/project-Getting+started+with+RTL-SDR) around getting started with the rtl-sdr.

This tool is _"a simple FFT logger"_:
```console
pi@raspberrypi:~ $ rtl_power -h
rtl_power, a simple FFT logger for RTL2832 based DVB-T receivers

Use:	rtl_power -f freq_range [-options] [filename]
	-f lower:upper:bin_size [Hz]
	 (bin size is a maximum, smaller more convenient bins
	  will be used.  valid range 1Hz - 2.8MHz)
	[-i integration_interval (default: 10 seconds)]
	 (buggy if a full sweep takes longer than the interval)
	[-1 enables single-shot mode (default: off)]
	[-e exit_timer (default: off/0)]
	[-d device_index (default: 0)]
	[-g tuner_gain (default: automatic)]
	[-p ppm_error (default: 0)]
	[-T enable bias-T on GPIO PIN 0 (works for rtl-sdr.com v3/v4 dongles)]
	filename (a '-' dumps samples to stdout)
	 (omitting the filename also uses stdout)
...
```

> and there is further discussion of it on the rtl-sdr blog including applicability in
> -  [RF fingerprinting](https://www.rtl-sdr.com/fingerprinting-electronic-devices-via-their-rf-emissions-with-an-rtl-sdr-and-imagemagick/)
> - [RF Power scans](https://www.rtl-sdr.com/combining-android-tasker-and-an-rtl-sdr-for-mobile-automated-frequency-power-scans/)
> - and a [UI Tool called `rfSpectrum`](https://www.rtl-sdr.com/rtlspectrum-a-new-gui-for-rtl_power/). 
{: .prompt-tip }


Running a quick scan of the FM Broadcast channels per Charlies Labs:
```shell
rtl_power -f 95M:105M:100k -i 10 -1 > fmscan.csv
``` 

![Image from heatmat.py script run on rtl_power output](31-fmscan-heat-out.png){: width="129" height="27" .w-50 .right}
There is output from that command, but more reading will be needed to know what to do with it so plenty more to explore! I did briefly grab one of their scripts called `heatmap.py`, but the output image(right) was not ideal, tweaking required...


## Cutting down on power use of the PI
I kind of considered this would be a pre-requisite, but things are going good. I will go ahead anyway with HDMI and Wireless disable in the hope that it will help the RF quality on the RTL-SDR, at least it shouldn't hurt.

I have already updated the raspberry [to turn off things I know I wont' use per above](#raspberry-configuration-customisations)
    

### Boot config controlled changes
As discussed for the Pi B, [these notes on reducing power use (2021)](https://blues.com/blog/tips-tricks-optimizing-raspberry-pi-power/) were on of the main inspirations.

Looking through `/boot/config.txt` which was there for the "Stretch" version of Debian, it has been moved to `/boot/firmware/config.txt` in the "Bookwork" OS version,, and appears to have a lot less in it.

This in turn references to `/boot/firmware/overlays/README` (moved from `/boot/overlays/README` in "Stretch"). 
Lucky I had a look at that, it refers to what appear to be newer overlay options for the RPI 3, so i updated a few as follows:
```config
# lines updated, ei2081-custom
camera_auto_detect=0
display_auto_detect=0

#  Broadcom VideoCore 4 contains a OpenGL ES 2.0-compatible 3D engine called V3D, and a highly configurable display output pipeline that supports HDMI, DSI, DPI, and Composite TV output. 
#dtoverlay=vc4-kms-v3d
#max_framebuffers=2
# ei2081-custom commented above two lines

# ei2081-custom lines added after [all]
dtoverlay=disable-wifi
dtoverlay=disable-bt
```
I applied those with the "pico" editor: `pi@raspberrypi:~ $ sudo pico /boot/config.txt`

### Cli controlled changes
I had a note to disable HDMI with `tvservice -o` (which worked for Stretch) but it's not there in this OS.  Neither is `rpi-hdmi`. 
I did find somewhere mention of `sudo dtparam hdmi=off` which doesnt' appear to do anything. Other tools like  [`cec-client`](https://pimylifeup.com/raspberrypi-hdmi-cec/) and `ddcutil` were also mentioned but I didn't get far with those. 

A suggestion to use `vcgencmd display_power 0` does the trick though. And immediately. When I use it my display/screen reports the same message as when i unplug the HDMI cable. [The source post](https://raspberrypi.stackexchange.com/questions/145461/turn-off-hdmi-rpi-4b-bookworm) also highlights that we can use parameter `1` to turn it back on.

> Be aware, if you turn off HDMI, you have limited options for recovering from errors if the Pi stops responding to your network, remote `ssh` requests, etc. I encountered such a problem and was able to sort it out by editing OS config on the card from a linux machine - but that may not be for everyone!
{: .prompt-warning }

## Summary
So now we appear to have a good driver install and should be all set to get into some more insteresting things with the RTL SDR Blog V4 and the Raspberry Pi 3. To be continued...

## Appendices
### Appendix A. Build prereqs install output
```console
pi@raspberrypi:/boot/firmware/overlays $ sudo apt-get install libusb-1.0-0-dev git cmake pkg-config 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
libusb-1.0-0-dev is already the newest version (2:1.0.26-1).
git is already the newest version (1:2.39.2-1.1).
cmake is already the newest version (3.25.1-1).
pkg-config is already the newest version (1.8.1-1).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

[Back](#getting-driver-build-pre-requisites)


### Appendix B. Build config prep output
```console
pi@raspberrypi:~/devel $ 
    cd rtl-sdr-blog
    mkdir build 
    cd build 
    cmake ../ -DINSTALL_UDEV_RULES=ON 

-- The C compiler identification is GNU 12.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Build type not specified: defaulting to release.
-- Extracting version information from git describe...
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Success
-- Found Threads: TRUE  
-- Found PkgConfig: /usr/bin/pkg-config (found version "1.8.1") 
-- Checking for module 'libusb-1.0'
--   Found libusb-1.0, version 1.0.26
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
-- Build files have been written to: /home/pi/devel/rtl-sdr-blog/build
```

[Back](#grabbing-the-code-and-preparing-it-for-build)

### Appendix C. Build make output
```console
pi@raspberrypi:~/devel/rtl-sdr-blog/build $ make
[  3%] Building C object src/CMakeFiles/rtlsdr.dir/librtlsdr.c.o
[  6%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_e4k.c.o
[  9%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_fc0012.c.o
[ 12%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_fc0013.c.o
[ 15%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_fc2580.c.o
[ 18%] Building C object src/CMakeFiles/rtlsdr.dir/tuner_r82xx.c.o
[ 21%] Linking C shared library librtlsdr.so
[ 21%] Built target rtlsdr
[ 25%] Building C object src/CMakeFiles/rtlsdr_static.dir/librtlsdr.c.o
[ 28%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_e4k.c.o
[ 31%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_fc0012.c.o
[ 34%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_fc0013.c.o
[ 37%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_fc2580.c.o
[ 40%] Building C object src/CMakeFiles/rtlsdr_static.dir/tuner_r82xx.c.o
[ 43%] Linking C static library librtlsdr.a
[ 43%] Built target rtlsdr_static
[ 46%] Building C object src/CMakeFiles/convenience_static.dir/convenience/convenience.c.o
[ 50%] Linking C static library libconvenience_static.a
[ 50%] Built target convenience_static
[ 53%] Building C object src/CMakeFiles/rtl_sdr.dir/rtl_sdr.c.o
[ 56%] Linking C executable rtl_sdr
[ 56%] Built target rtl_sdr
[ 59%] Building C object src/CMakeFiles/rtl_tcp.dir/rtl_tcp.c.o
[ 62%] Linking C executable rtl_tcp
[ 62%] Built target rtl_tcp
[ 65%] Building C object src/CMakeFiles/rtl_test.dir/rtl_test.c.o
[ 68%] Linking C executable rtl_test
[ 68%] Built target rtl_test
[ 71%] Building C object src/CMakeFiles/rtl_fm.dir/rtl_fm.c.o
[ 75%] Linking C executable rtl_fm
[ 75%] Built target rtl_fm
[ 78%] Building C object src/CMakeFiles/rtl_eeprom.dir/rtl_eeprom.c.o
[ 81%] Linking C executable rtl_eeprom
[ 81%] Built target rtl_eeprom
[ 84%] Building C object src/CMakeFiles/rtl_adsb.dir/rtl_adsb.c.o
[ 87%] Linking C executable rtl_adsb
[ 87%] Built target rtl_adsb
[ 90%] Building C object src/CMakeFiles/rtl_power.dir/rtl_power.c.o
[ 93%] Linking C executable rtl_power
[ 93%] Built target rtl_power
[ 96%] Building C object src/CMakeFiles/rtl_biast.dir/rtl_biast.c.o
[100%] Linking C executable rtl_biast
[100%] Built target rtl_biast
```

[Back](#build-was-the-same)


### Appendix D. Install Output
```console
pi@raspberrypi:~/devel/rtl-sdr-blog/build $ sudo make install
[ 21%] Built target rtlsdr
[ 43%] Built target rtlsdr_static
[ 50%] Built target convenience_static
[ 56%] Built target rtl_sdr
[ 62%] Built target rtl_tcp
[ 68%] Built target rtl_test
[ 75%] Built target rtl_fm
[ 81%] Built target rtl_eeprom
[ 87%] Built target rtl_adsb
[ 93%] Built target rtl_power
[100%] Built target rtl_biast
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
pi@raspberrypi:~/devel/rtl-sdr-blog/build $ sudo cp ../rtl-sdr.rules /etc/udev/rules.d/ 
pi@raspberrypi:~/devel/rtl-sdr-blog/build $ sudo ldconfig
pi@raspberrypi:~/devel/rtl-sdr-blog/build $ 
```

[Bacl](#install-of-the-built-driver-rules-and-dynamic-linker)


### Appendix E. Blacklist DVB-T
```console
pi@raspberrypi:~/devel/rtl-sdr-blog/build $ echo 'blacklist dvb_usb_rtl28xxu' | sudo tee --append /etc/modprobe.d/blacklist-dvb_usb_rtl28xxu.conf blacklist dvb_usb_rtl28xxu 
blacklist dvb_usb_rtl28xxu
```

[Back](#blacklisting-dvb-ttv-drivers)


### Appendix F. Monitor output for Device Connect
From the log/`journalctl`:
```console

Aug 31 12:50:53 raspberrypi kernel: usb 1-1.2: new high-speed USB device number 5 using dwc_otg
Aug 31 12:50:53 raspberrypi kernel: usb 1-1.2: New USB device found, idVendor=0bda, idProduct=2838, bcdDevice= 1.00
Aug 31 12:50:53 raspberrypi kernel: usb 1-1.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
Aug 31 12:50:53 raspberrypi kernel: usb 1-1.2: Product: Blog V4
Aug 31 12:50:53 raspberrypi kernel: usb 1-1.2: Manufacturer: RTLSDRBlog
Aug 31 12:50:53 raspberrypi kernel: usb 1-1.2: SerialNumber: 00000001
Aug 31 12:50:54 raspberrypi mtp-probe[878]: checking bus 1, device 5: "/sys/devices/platform/soc/3f980000.usb/usb1/1-1/1-1.2"
Aug 31 12:50:54 raspberrypi mtp-probe[878]: bus: 1, device: 5 was not an MTP device
Aug 31 12:50:54 raspberrypi mtp-probe[880]: checking bus 1, device 5: "/sys/devices/platform/soc/3f980000.usb/usb1/1-1/1-1.2"
Aug 31 12:50:54 raspberrypi mtp-probe[880]: bus: 1, device: 5 was not an MTP device
```

[Back](#monitoring-the-pi)

From kernel/`dmesg`
```console

[  673.821602] usb 1-1.2: new high-speed USB device number 5 using dwc_otg
[  673.933737] usb 1-1.2: New USB device found, idVendor=0bda, idProduct=2838, bcdDevice= 1.00
[  673.933778] usb 1-1.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[  673.933795] usb 1-1.2: Product: Blog V4
[  673.933807] usb 1-1.2: Manufacturer: RTLSDRBlog
[  673.933819] usb 1-1.2: SerialNumber: 00000001

```
