---
title: Raspberry Pi Models
description: 
date: 2024-08-28 02:00:01 +0000
categories: [hardware, µP]
tags: [raspberrypi, debian] 
media_subpath: /assets/img/2024/08/
image:
    path: 28-pi-b-top.png
    alt: Picture of the Raspberry P Model B Rev 2
---

# Raspberry PI Models

We have two Pi micro computers (or "single board computers" ) right now which we plan to try with the RTL SDR Blog V4.
- [Pi Model B](#raspberry-pi-model-b-rev2)
- [Pi 3 Model B+](#raspberry-pi-3-b-r13)

## Raspberry PI Model B Rev2 

The first is an old 2011 _"Raspberry PI Rev 2 Model B"_. Over the years it has run a demo website with dynamic dns serving, a media player and some retro games. Various operating system have graced its memory cards including Raspian, Debian, Arch and even RISCOS. 

It may not be up to spec to run the [RTL SDR V4]({% post_url 2024-08-27-rtl-sdr-blog-v4 %}) but we will have a go.

At the time of writing there are two cards for it with installs that haven't been touched in a while, updates may well be on the cards. They are both running Debian `Raspbian GNU/Linux 10 (buster)`. One with a UI and one without, terminal only. Specific version via `cat /etc/debian_version` = `10.13`


### Hardware Ref
This model is one of the earlier Raspberry Pi devices. Initially I couldn't find it on the Pi site - it does not show up on [the RaspberryPi.com products page](https://www.raspberrypi.com/products/). But it does show up [on this documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html) page.

### PCB Markings/Text
![Picture of the RTL-SDR Blog V4 device](28-pi-b-top.png){: width="320" height="215" .right}
on the top of the board, text other than component ids/numbers includes
- under the Raspberry logo
    > "Raspberry Pi (c) 2011.12"
- a sticker at the edge
    > "Made in China" 
- Not much else, icons for the "CE" and "FCC" logos.

![Picture of the RTL-SDR Blog V4 device](28-pi-b-btm.png){: width="320" height="215"  .right}
On the back of the board there is an "SL" logo text, and well as
- "E234156"
- "SL-M"
- "RL 94V-0"
- and a sticker with `E4512RS2V13B1.0`

### Mounted Ports
- 1x usb micro-b, power
- 1x sd card
- 26x gpio pins
- 1x composite video out (yellow) RCA
- 1x audio out
- 2x USB A 2.0
- 1x lan 100Mb/s
- CDI camera port
- DSI display port
- 1x HDMI

### Ether Ports
There is no wifi nor bluetooth

### Model specifics from OS
A file which contains the Pi's model. (on some operating systems):

```console
> cat /sys/firmware/devicetree/base/model
Raspberry Pi Model B Rev 2
```

### CPU detail from OS
`lscpu` says:
```console
> lscpu
Architecture:        armv6l
Byte Order:          Little Endian
CPU(s):              1
On-line CPU(s) list: 0
Thread(s) per core:  1
Core(s) per socket:  1
Socket(s):           1
Vendor ID:           ARM
Model:               7
Model name:          ARM1176
Stepping:            r0p7
CPU max MHz:         700.0000
CPU min MHz:         700.0000
BogoMIPS:            697.95
Flags:               half thumb fastmult vfp edsp java tl
```
### Memory detail from OS
`free` says about memory:
```console
> free -h
total        used        free      shared  buff/cache   available
Mem:          432Mi        31Mi        66Mi       5.0Mi       334Mi       339Mi
Swap:          99Mi       1.0Mi        98Mi
```



## Raspberry PI 3, B+, R1.3

Our second is a _"Raspberry PI 3, B+, R1.3"_. It has 3 more CPU cores and twice as much ram compared to the Pi model B and some extra peripherals.

### Hardware Ref
More on [the raspberry products page](https://www.raspberrypi.com/products/) and detail is [there at this direct link](https://www.raspberrypi.com/products/raspberry-pi-3-model-b-plus/).

### PCB Markings / Text 

On the top of the board there is ...
- "Raspberry Pi 3 Model B+"
- "(c) Raspberry Pi 2017"
- the pi Logo

On the back of the board there is ...
- "CE" & "FCC" logos
- "IC 20953-RPI3P"
- "FCC ID: 2ABCB-RPI-3BP"
- "KCE MC1"
- "94V-0 F1"
- "1818 B1"


### Mounted Ports
- 1x usb micro-b, power
- 1x micro-sd card
- 40x gpio pins
- 1x audio out
- 4x USB A, 2.0
- 1x lan with poe, 300Mb/s
- CSI camera port
- DSI display port
- 1x HDMI

### Ether Ports
- dual-band wireless LAN 100Mb/s 802.11ac
- Bluetooth 4.2/BLE low energy

### Model detail from the OS
A file which contains the Pi's model. (on some operating systems):

```console
> cat /sys/firmware/devicetree/base/model
Raspberry Pi 3 Model B Plus Rev 1.3
```

### CPU detail from the os
`lscpu` says:
```console
> lscpu
Architecture:          armv7l
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    4
Socket(s):             1
Model:                 4
Model name:            ARMv7 Processor rev 4 (v7l)
CPU max MHz:           1400.0000
CPU min MHz:           600.0000
BogoMIPS:              38.40
Flags:                 half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
```

### Memory
`free` says about memory:
```console
> free -h
              total        used        free      shared  buff/cache   available
Mem:           927M        206M        343M         43M        377M        639M
Swap:           99M          0B         99M
```


