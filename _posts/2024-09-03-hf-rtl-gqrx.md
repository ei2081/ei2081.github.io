---
title: Brief digimode test with a V4 RTL-SDR and a random wire
description: Testing digital mode reception on the HAM bands with an RTL-SDR Blog V4, GQRX and WSJTx - a some scrap wire
date: 2024-09-01 00:00:01 +0000
categories: [projects, rtlsdr]
tags: [rtl-sdr, raspberrypi, sdr, ft8, ft4, randomwire]
media_subpath: /assets/img/2024/09/
image:
    path: 03a-randwire.png
    alt: a literal random bit of wire wound on a bit of 2x1
---

## Random wire test with GQRX, WSJTx and the RTL SDR V4
![Picture of the cord](03b-cord.png){: width="200" height="240" .right} 
For a few days now I've being trying - here and there - to catch any WSPR, FT8, FT4 or other digital modes on the HAM bands with the [RTL-SDR Blog V4 and GQRx]({% post_url 2024-08-27-rtl-sdr-blog-v4 %}). The [dipole kit](https://www.rtl-sdr.com/dipole) which I got has an antenna set which covers from about 70MHz (4m band) to about 1030MHz (30cm ish?) out of the box. So, within those frequencies is where I've been focused, and all I have managed to identfy so far was some APRS chatter on 2m, which I will look back at in future, and some other digital sounding modes which I didn't identify. 

There has been nothing that I can find in terms of an FT* mode or WSPR aroudn the frequencies you'd expect to find them from looking at the band plans. I do see a few high frequency reports on HAM Alert for 23cm or higher, but only for short windows here and there over the week. Looking at the [PSK Reporter](https://pskreporter.info) there is some FT* monitoring on 2m for example, but no recorded traffic of late in my vicinity.

So, I decided to have a go at the lower frequencies where I know - from experience with Web SDRs and local HAM stations - there is regular traffic and often better propogation. After scouring the house I eventually found ca. 8m wire that I hope might do the job. 
![Picture of the coax connected to the wire](03c-connection.png){: width="225" height="220" .left}
It was single core and insulated - plastic covered, no rf shielding.

This was to be a quick and dirty test, so I just strung it down the garden, with some old Kite cord to fix it to a fence post. 

The other end was just brought in the window, held with a something heavy and inserted into the centre connection of the coax I got with the SDR Dipole kit! That coax then runs directly to the RTL-SDR which is connected to my laptop.

## Bands covered
I decided to try various frequencies on the 10, 12, 15, 17 and 20m bands for both FT8 and FT4. I also tried the WSPR frequencies, and while I was sure I could sometimes hear a faint WSPR-like warble in the noise, nothing was detected by WSJTx so I will skip that for now.
![Picture of gqrx on 14.074](03d-gqrx.png){: width="225" height="220" .right}

WSJTx doesn't currently have any support for the RTL-SDR out of the box, so I had wondered about getting the audio to it. For instance, I had suppossed there may be an option with GQRX connected to the RTL, passing the audio on to the WSJTx application through some clever combination of software utilities.

As I was wondering about this, I recalled my first use of WSJTx and the FT* mode with a real rig in July. Before then I had only ever monitored FT8/4 on Web SDRs. I was at a licenced Amateurs station, with my laptop, WSJTx and an audio cable. We tuned to a busy FT4 frequency and I prepared to connect my laptop to the radios audio feed. However, I had started WSJTx first, and the suprise was that it immediately began decoding the signal it was able to hear through the laptops micropone, no cable connections required :D

With that in mind, I now wondered if the same might apply in this scenario. Could I use GQRX receiving from the RTL-SDR, have it playing FT4/8 audio over the laptop speakers, and _on the same laptop_ have WSJTx listening on the microphone and successfully decode it.... ?

###  Steps
1. Start WSJTx 
1. Pick a mode and frequency. 
1. Click stop so it doesn't start decoding and reporting right away 
  - Without a CAT controlled rig, it is very easy for this to happen. 
  - WSJTx will start reporting to PSK Reporter what it hears.
  - So if WSJTx is not set to the same frequency as the rig, decodes will be assigned to the wrong band
1. Tune GQRX to that freq, and if you even hear a faint warble, hit monitor on WSJTx
1. WSJTx listening to the mic can pick up GQRX playing to the speakers, no intra-application magic needed. :D
1. When your happy, stop GQRX from playing sound, then go back to step 2 to try another mode/freq.

![Picture of wsjt decodes](03e-wsjtx.png){: width="225" height="220" }

### Experience
I tried these bands / frequencies. 20m was fab, very clear as soon as I tuned. 
On 17 and 15 I could hear some faint signals intermittantly, and WSJTx picked up a couple on 17m, but there was an awful lot of noise. The other bands were mostly noise.

|  Band | Mode     | Freq        | Heard? |  
|-------|----------|-------------|--------|
|  20m  | FT8      | 14.074 MHz  | 32  |  
|  20m  | FT4      | 14.080 MHz  | 10  | 

|  17m  | FT8      | 18.100 MHz  | 2   | 
|  17m  | FT4      | 18.104 MHz  | 0   | 

|  15m  | FT8      | 21.074 MHz  | 0   |    
|  15m  | FT4      | 21.091 MHz  | 0   |

|  12m  | FT8      | 24.915 MHz  | 0   |    
|  12m  | FT4      | 24.919 MHz  | 0   |

|  10m  | FT8      | 28.000 MHz  | 0   | 
|  10m  | FT4      | 28.000 MHz  | 0   | 

## Contact !
On 20m signals came in quite clearly and the WSJTx logs started filling up readily. On 17m things were tainted with noise but one could pick out clear signals with a bit of difficulty and WSJTx even decoded a pair of them in the short window I left things active. At the time the other bands 15, 12, and 10 were just destroyed with noise. Whether that be conditions or something about my setup will be something I'm sure I'll figure out eventually.

![Picture of psk heards](03f-psk.png){: width="225" height="220" .right}

For now, this was an interesting excercise in what may be achievable below 70MHz with no prepratation whatsoever, and it was a massive suprise in terms of what one can do with a bit of random wire. For instance, picking up South American stations, the Carribean, North America and even Eastern China were a big shock given the setup!

## Appendix - List of stations reported

This is the list of stations reported to PSKReporter during the setup window.

### 17m

| Freq      | Dist   | Mode | Date     | Time   | SNR | Call    | Country | DXCC | Grid   |
| ----      | ----   | ---- | ----     | ----   | --- | ----    | ----    | ---- | ----   |
| 18.100788 | 4322.8 | FT8  | 20240903 | 220229 | -1  | VE1KCO  | Canada  | 1    | FN64XJ | 
| 18.101615 | 1343.8 | FT8  | 20240903 | 220329 | -24 | IU1NOD  | Italy   | 248  | JN34SX |   


### 20m

| Freq      | Dist   | Mode | Date     | Time   | SNR | Call    | Country            | DXCC | Grid   |
| ----      | ----   | ---- | ----     | ----   | --- | ----    | ----               | ---- | ----   |
| 14.074512 | 6401.5 | FT8  | 20240903 | 213127 | -8  | KP4QVQ  | Puerto Rico        | 202  | FK68KD   | 
| 14.075434 | 5598.1 | FT8  | 20240903 | 213127 | -17 | AD8IO   | United States      | 291  | EN81UF   | 
| 14.075319 | 1833.0 | FT8  | 20240903 | 213127 | 0   | HA1BLA  | Hungary            | 239  | JN87TQ   | 
| 14.075720 | 4039.5 | FT8  | 20240903 | 213128 | -22 | 4L4DX   | Georgia            | 75   | LN21LN   | 
| 14.075781 | 2096.7 | FT8  | 20240903 | 213130 | -6  | E75C    | Bosnia-Herzegovina | 501  | JN93DT   | 
| 14.075992 | 2768.7 | FT8  | 20240903 | 213131 | -2  | YO4NF   | Romania            | 275  | KN44hd   | 
| 14.075056 | 5279.7 | FT8  | 20240903 | 213131 | -17 | VE3NYZ  | Canada             | 1    | FN03ia55   | 
| 14.075120 | 2047.0 | FT8  | 20240903 | 213131 | -3  | IZ8DBJ  | Italy              | 248  | JN70at55aa   | 
| 14.075216 | 2671.1 | FT8  | 20240903 | 213132 | -12 | LZ1KGR  | Bulgaria           | 212  | KN22TK   | 
| 14.075146 | 4992.6 | FT8  | 20240903 | 213133 | -11 | VA3BOO  | Canada             | 1    | FN14vh87   | 
| 14.075812 | 1755.7 | FT8  | 20240903 | 213133 | -8  | S50KGB  | Slovenia           | 499  | JN76TN   | 
| 14.076225 | 1845.8 | FT8  | 20240903 | 213141 | 8   | SP9UXV  | Poland             | 269  | JO90pe   | 
| 14.075975 | 2260.8 | FT8  | 20240903 | 213141 | -6  | UR5SDL  | Ukraine            | 288  | KN28IW   | 
| 14.074701 | 1736.2 | FT8  | 20240903 | 213141 | -3  | EA7HTE  | Spain              | 281  | IM66VP   | 
| 14.074471 | 8524.1 | FT8  | 20240903 | 213142 | -13 | PT2BW   | Brazil             | 108  | GH64bg34ab | 
| 14.075031 | 8703.7 | FT8  | 20240903 | 213142 | -7  | PY4HI   | Brazil             | 108  | GH80PK   | 
| 14.075375 | 2360.6 | FT8  | 20240903 | 213142 | -2  | IK8IRK  | Italy              | 248  | JM88aj   | 
| 14.074514 | 2443.2 | FT8  | 20240903 | 213145 | -18 | RA1AIR  | European Russia    | 54   | KP50ea   | 
| 14.074891 | 2557.4 | FT8  | 20240903 | 213145 | -16 | LZ2VQ   | Bulgaria           | 212  | KN23IF14CE | 
| 14.074969 | 1987.6 | FT8  | 20240903 | 213145 | -13 | SP8TN   | Poland             | 269  | KO00RI   | 
| 14.076310 | 4773.5 | FT8  | 20240903 | 213145 | -11 | WB1EVU  | United States      | 291  | FN34ML   | 
| 14.076450 | 5245.0 | FT8  | 20240903 | 213146 | -18 | W3FRG   | United States      | 291  | FM29GV   | 
| 14.076419 | 3063.0 | FT8  | 20240903 | 213147 | -14 | UX2LM   | Ukraine            | 288  | KN89LM   | 
| 14.074595 | 1983.4 | FT8  | 20240903 | 213156 | 7   | 9A2TN   | Croatia            | 497  | JN95BL   | 
| 14.076097 | 9340.9 | FT8  | 20240903 | 213157 | 1   | BG4UCZ  | China              | 318  | PM02BC   | 
| 14.075631 | 6510.1 | FT8  | 20240903 | 213200 | -24 | W4MRJ   | United States      | 291  | EL98aw63   | 
| 14.075583 | 6837.1 | FT8  | 20240903 | 213201 | -24 | CO6XDX  | Cuba               | 70   | FL02LM   | 
| 14.074435 | 6556.6 | FT8  | 20240903 | 213212 | -8  | HI8WJI  | Dominican Republic | 72   | FK58BL   | 
| 14.074742 | 6032.0 | FT8  | 20240903 | 213212 | -8  | K4CAE   | United States      | 291  | EM94NC   | 
| 14.075105 | 5346.2 | FT8  | 20240903 | 213216 | 6   | N4ZR    | United States      | 291  | FM19rl   | 
| 14.075851 | 2240.9 | FT8  | 20240903 | 213216 | -11 | ES5AI   | Estonia            | 52   | KO38   | 
| 14.075187 | 1138.5 | FT8  | 20240903 | 213217 | -20 | DM2JJ   | Germany            | 230  | JN49GM   | 
| 14.082479 | 2084.2 | FT4  | 20240903 | 213243 | 11  | OH6FSO  | Finland            | 224  | KP12eo72   | 
| 14.080937 | 1694.2 | FT4  | 20240903 | 213328 | 1   | CT2FZZ  | Portugal           | 272  | IM67aa   | 
| 14.082012 | 2096.7 | FT4  | 20240903 | 213336 | 13  | E75C    | Bosnia-Herzegovina | 501  | JN93DT   | 
| 14.080930 | 2791.3 | FT4  | 20240903 | 213336 | 15  | EG8SDC  | Canary Islands     | 29   | IL18NF   | 
| 14.082610 | 1900.7 | FT4  | 20240903 | 213336 | -13 | SP4JAW  | Poland             | 269  | KO04QB   | 
| 14.081028 | 6049.2 | FT4  | 20240903 | 213343 | -8  | K4SUE   | United States      | 291  | FM02CW   | 
| 14.080301 | 5070.7 | FT4  | 20240903 | 213343 | -4  | K2FJ    | United States      | 291  | FN21xb52   | 
| 14.080983 | 6519.6 | FT4  | 20240903 | 213445 | -7  | NA6Y    | United States      | 291  | EL97sm97   | 
| 14.082395 | 2573.4 | FT4  | 20240903 | 214643 | -8  | YO3GNF  | Romania            | 275  | KN34al58   | 
| 14.081306 | 2259.7 | FT4  | 20240903 | 214836 | 6   | CT3MD   | Madeira Islands    | 256  | IM13tb73hu  | 

