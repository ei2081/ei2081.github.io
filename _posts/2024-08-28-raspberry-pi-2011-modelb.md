---
title: Raspberry Pi Models - Model B, Rev2
description: 
date: 2024-08-28 02:00:01 +0000
categories: [hardware, µP]
tags: [raspberrypi, debian] 
media_subpath: /assets/img/2024/08/
image:
    path: 28-pi-b-top.png
    alt: Picture of the Raspberry P Model B Rev 2
---

# Raspberry PI Rev2 Model B

We have an old 2011 "Raspberry PI Rev 2 Model B" but it's not done much over the years to be quite honest. It did run a demo website with dynamic dns serving, a media player and some retro games. Various operating system have graced it's memory cards including Raspian, Debian, Arch and even RISCOS. 

I'm not sure if it's up to spec to run the [RTL SDR V4]({% post_url 2024-08-27-rtl-sdr-blog-v4 %}) but will have a go.

At the time of writing there are two cards for it with installs that haven't been touched in a while, updates may well be on the cards.

1. a 2GB μSD with μSD to SD adaptor. 
    - Debian
    - `Raspbian GNU/Linux 10 (buster)` 
    - without a UI, terminal only
    - `cat /etc/debian_version` = `10.3`
1. a 4GB SD. 
    - Debian 
    - `Raspbian GNU/Linux 10 (buster)` 
    - LXDE as the UI.
    - `cat /etc/debian_version` = `10.13`

I'd note there's not a lot of space on the 2GB even with just the most basic of installs. For something very specific it's totally serviciable, but doesn't suit well if I need to make a build due to all the tooling and dependancy requirements chewing up the space. 4GB+ will make things a lot more comfortable.

## Hardware 
This model is one of the earlier Raspberry Pi devices. Initially I couldn't find it on the Pi site - it does not show up on [the RaspberryPi.com products page](https://www.raspberrypi.com/products/). But it does show up [on this documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html) page.

### PCB Text Notes
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

## OS Hardware Info
This is what the install of Debian Buster is ableto tell us about the board.

### Model
A file which contains the Pi's model. (on some operating systems):

```console
> cat /sys/firmware/devicetree/base/model
Raspberry Pi Model B Rev 2
```

### CPU
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
### Memory
`free` says about memory:
```console
> free -h
total        used        free      shared  buff/cache   available
Mem:          432Mi        31Mi        66Mi       5.0Mi       334Mi       339Mi
Swap:          99Mi       1.0Mi        98Mi
```


