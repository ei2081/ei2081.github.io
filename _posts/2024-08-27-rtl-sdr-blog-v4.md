---
title:  First use of an RTL-SDR BLOG V4
description: You have mail! Initial tests with an RTL-SDR BLOG V4 
date: 2024-08-27 21:00:01 +0000
categories: [hardware, Rx]
tags: [sdr, rtl-sdr] 
media_subpath: /assets/img/2024/08/
image:
    path: 27-rtl-sdr.png
    alt: Picture of the RTL-SDR Blog V4 device
---

# This past month I received delivery of...
![Picture of the RTL-SDR Blog V4 device](27-rtl-sdr.png){: width="206" height="90" .w-50 }
... an [RTL-SDR BLOG V4 kit complete with a dipole antenna](https://www.rtl-sdr.com/dipole/). I have no HF gear right now, so this is an opportunity move beyond 70cm/2m & AM/FM modes, excluding the us of Web SDRs from locations around the globe ([ref](#web-sdr-references)).

## Drivers
On some systems, driver work is needed for the V4, but for the OSx options the driver already comes as part of the various app packages. 

The notes on compiling the driver for various systems are available from [their github](https://github.com/rtlsdrblog/rtl-sdr-blog) should you need it, and I want to use this device with a Raspberry Pi so I'll be coming back to that myself.


##  Software
I had a look at some of the software clients mentioned on the [RTL-SDR BLOG](https://www.rtl-sdr.com/V4/) for this device, and filtered that for ones that run on MacOS. (You'll have to scroll their site to get to the relevant section, at time of writing the articles don't have toc or links direct to sections.)

Here are the applications I tried:
1. GQRX
    - Download size ca. `55MB` at `v2.17.5` (2024-04-18 build)
    - installable via [Brew](https://formulae.brew.sh/cask/gqrx). (`brew install --cask gqrx` on intel mac)  
    - On [github](https://github.com/gqrx-sdr/gqrx?tab=readme-ov-file).
1. SDR++
    - Download size ca. `7MB` at `v1.2.0` (2024-08-27 build)
    - dmg via [github](https://github.com/AlexandreRouma/SDRPlusPlus/releases)
    - makes the CPU work hard even without dongle !
1. SDR Angel
    - `150MB` at `v7.22.0` (2024-08-13 build)
    - dmg via [github](https://github.com/f4exb/sdrangel/releases).



## Trial Experience
Aside from the GQRX Brew install, all of the applications result in warnings from OSx that the application is not verified and could contain malware. There are steps one can take to run them anyway...

### Summary
I did have a good look around the bands, from 70MHz up to 1.7GHz (the range the dipole covers). There was a lot of broadcast and voice activity on the ham, broadcast and marine bands where you'd expect it. I also encountered various some digital modes. I did not find traffic on the expected frequencies for WSPR or FT* modes yet on UHF/VHF. I may try again as soon as I secure an antenna for below 70MHz...


### GQRX
Install with brew was simple and fast.

![Picture of GQRX in actio on 4m](27-rtlsdr-gqrx-4m.png){: width="240" height="320" .w-50 .right}
I got the "configure i/o devices" windows as expected, after starting from terminal via `gqrx`, and minor config only is required:

- selected `RTLSDRBlog Blog V4 SN...`
- selected output audio device to my headphones

It spun up very quickly and going to the FM Audio bands i can see immediately an improvement. Where with SDR++ only local radio showed, now i see national, e.g. RNaG on 93.4MHz and the [EI2HHR 4m repeater on 70.4MHz](https://www.irts.ie/cgi/repeater.cgi) connected to the [SIRN network](https://sirnrepeaters.blogspot.com/) (see image).

There was no sign of the CPU fan kicking in or general pc slow down (it was variable 25-75% cpu), but of course we can alter the settings like the FFT to make it do more work and change that.

Their [blog](https://www.gqrx.dk) has more detail including remote freq control, audio stream to other programs, etc...


### SDR Plus Plus
Once running (see note above re malware warnings) it's a matter of attaching the RTL-SDR, selecting the device under the "Source" section and clicking refresh, and then clicking the big play button with the usual triangle icon.

FM Spikes and audio at the expected location for Local Radio.

I didnt' get much luck with the national stations.

It seems a bit more featured than GQRX from a ui perspective, but unfortunately I didn't experience the same level of receptions, and the pc fan spun up and everything became quite slow on the laptop, even when running with the SDR off.

Having a look at the osx activity monitor while it's on, it shows a constant 102% cpu with SDRpp on, drops to 25% & less after killing it.


### SDR Angel
Once running (see note above re malware warnings), SDR Angel also requests to 
- use the camera
- use my location
- accept incoming network connections

![Picture of the Dipole Antenna in the Window](27-rtl-sdr-dipole.png){: width="240" height="320" .w-50 .left}
The UI didn't initially appear to have a lot going on, but once I found the configuration options and tried a few defaults - I started to see data coming in on the european APRS at `144.800` for various calls around the country, and the transmissions were decoded which was kinda cool.

The combination of saved defaults for various configurations, coupled with demodulators etc seems very exciting to use. 
I did get a 404 error for the ISS packet config, and I couldn't get the audio working. 

The requests for access to camera & location continued after restart, so this one is removed from my machine.


## Wrapping Up
This is just the start hopefully.

I do not expect to be using any desktop apps with this SDR on a regular basis.

As such, I did not spend a lot of time with any of the above apps, just a quick check to see what they were like and how the SDR sounded in use, and ensure the device, reception, etc were all ok before spending time building drivers and setting up a headless system.

Next up will be hopefully a Raspberry Pi with the RTL SDR Blog V4....

## Web Sdr References

- Kiwi Web SDRs: http://kiwisdr.com/public/
- [Rx/Tx Receiver List](https://rx-tx.info/table-sdr-points?title=&type=All&country=All&order=field_country_name&sort=asc&page=0)
- [Web SDR.org](https://www.websdr.org/)

