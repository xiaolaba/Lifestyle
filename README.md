# 27.145 MHz BOSE Lifestyle

Controlling 27.145 MHz BOSE Lifestyle with __ESP8266__ and __Raspberry Pi__

## User story

I want to tie my BOSE Lifestyle system into my smart home control setup like this:

![mute](https://cloud.githubusercontent.com/assets/2480569/21288755/751ef764-c48b-11e6-8fdb-449a02545bbd.gif)

The Remote Control RC9 for BOSE Lifestyle 3, 5, 8, 12 systems is using radio rather than infrared. Also, since these systems do not have other ways to control them, it is difficult to integrate them into home automation setups. Also, there are no replacement remotes available and even worn-down used original remotes go for absurd prices on eBay.

This is why I was curious how these devices could be controlled using the apparently proprietary radio signals.

## Research

From the manual, I knew that the device was using 27.145 MHz for the remote control. So I started by investigating the FCC database, searching for the manufacturer BOSE and the frequency 27.145 MHz. I did not find my exact model but others that suggested that BOSE was using 27.145 MHz pulses with on-off keying (OOK). According to Wikipedia, OOK denotes the simplest form of amplitude-shift keying (ASK) modulation that represents digital data at the presence or absence of a carrier wave. Some of the documents in the FCC database show sample pulse trains.

### Teardown of original remote

![rc9](https://cloud.githubusercontent.com/assets/2480569/21288808/e0188b60-c48c-11e6-96ac-b2933d63d2a1.jpg)
![raltron](https://cloud.githubusercontent.com/assets/2480569/21288809/e237bf4c-c48c-11e6-9115-039306378108.jpg)

 * BOSE PCB is __HCMK-C2X__
 * Main IC is __NEC D6121G 001 0207D3001__ ([data sheet](http://www.datenblatt-pdf.com/download.php?id=647260)), or ([local_data sheet](uPD6121.pdf))
 * Ceramic Resonator is __CSB 455J__ (430-519 kHz according to data sheet; needed by the NEC D6121G)
 * Crystal is __RALTRON 27.145-30 02 CYH 04__

### Measured with Saleae Logic on D6121G pin 7 = REM

On pin 7 = REM, there is for the ON/OFF key an infrared-like signal modulated with 38 kHz and 33% duty cycle (yellow channel 4 below). The same signal but _not_ modulated with 38 kHz can be seen at L2 near the transmitting antenna (orange channel 3 below).

If keys are pressed for a longer time, repetitions happen after 39 ms.

![bildschirmfoto 2016-12-16 um 23 12 31](https://cloud.githubusercontent.com/assets/2480569/21288353/3476534c-c481-11e6-8faf-3197cb1ed768.png)

try to understand further why forked & author was saying NEC2 protocol, take a look for screen captured above closely, manually decoded those signal, it turns out bit stream is that 186,85,76,/76, it was saying the ON/OFF button pressed with real BOSE remote, however, it is 32bytes & raw data bytes in LSB first, here is another written for BOSE remote code and thsoe key functions,  
![why 76 is for ON/OFF key](https://github.com/probonopd/irdb/blob/master/codes/Bose/Lifestyle/186%2C85.csv)  
![BOSE remote ON-OFF NEC2 procotol.JPG](BOSE%20remote%20ON-OFF%20NEC2%20procotol.JPG)  
.  
.  


Zoom in a bit to see the difference between the signal modulated with 38 kHz and 33% duty cycle (yellow channel 4 below) and unmodulated at L2 near the transmitting antenna (orange channel 3 below).

![bildschirmfoto 2016-12-16 um 23 14 51](https://cloud.githubusercontent.com/assets/2480569/21288355/35e160fa-c481-11e6-8af2-0f57fe814df4.png)

Wait, these signals look familiar. This is a variant of the well-known NEC infrared protocol that is used by many infrared-controlled devices. In fact, the signal above is `protocol = NEC2, device = 186, subdevice = 85, obc = 76`. But instead of sending the signals using a 940 nm infrared LED pulsating at 38 kHz, they are using an antenna pulsating at 27.145 MHz for the periods marked with "+" below. I used the awesome [IrScrutinizer](http://www.harctoolbox.org/IrScrutinizer.html) tool to verify my hypothesis by generating some NEC codes. Then I checked the irdb infrared database, one of the largest crowd-sourced, manufacturer-independent databases of infrared remote control codes on the web. And indeed, I found [this set of BOSE codes](https://github.com/probonopd/irdb/blob/master/codes/Bose/Lifestyle/186%2C85.csv). Using [IrScrutinizer](http://www.harctoolbox.org/IrScrutinizer.html), it was easy to calculate the "clean" (calculated rather than measured) codes for all BOSE commands.

Hence I assume that these are the valid signals in nanoseconds, with "+" being "27.145 MHz on", and "-" = off:

```
MUTE
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

VOLUME -
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

VOLUME +
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

AM/FM
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

SURROUND -
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

SURROUND +
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

VIDEO 2
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

VIDEO 1
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

AUX
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628

SKIP <<
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -38628

SKIP >>
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -38628

STOP
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -38628

2 SP
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -38628

ON/OFF
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -38628

5 SP
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -38628

3 SP
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -38628

TAPE
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -38628

CD
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -38628

PLAY
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -38628

PAUSE
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -38628

RANDOM
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -38628

NEXT DISC
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -38628

MUTE ALL
+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -38628
```

In other words:

```
MUTE: NEC2 Device: 186.85 Function: 1 S=85
VOLUME -: NEC2 Device: 186.85 Function: 2 S=85
VOLUME +: NEC2 Device: 186.85 Function: 3 S=85
AM/FM: NEC2 Device: 186.85 Function: 6 S=85
SURROUND -: NEC2 Device: 186.85 Function: 10 S=85
SURROUND +: NEC2 Device: 186.85 Function: 11 S=85
VIDEO 2: NEC2 Device: 186.85 Function: 13 S=85
VIDEO 1: NEC2 Device: 186.85 Function: 14 S=85
AUX: NEC2 Device: 186.85 Function: 15 S=85
SKIP <<: NEC2 Device: 186.85 Function: 24 S=85
SKIP >>: NEC2 Device: 186.85 Function: 25 S=85
STOP: NEC2 Device: 186.85 Function: 26 S=85
2 SP: NEC2 Device: 186.85 Function: 75 S=85
ON/OFF: NEC2 Device: 186.85 Function: 76 S=85
5 SP: NEC2 Device: 186.85 Function: 78 S=85
3 SP: NEC2 Device: 186.85 Function: 79 S=85
TAPE: NEC2 Device: 186.85 Function: 82 S=85
CD: NEC2 Device: 186.85 Function: 83 S=85
PLAY: NEC2 Device: 186.85 Function: 85 S=85
PAUSE: NEC2 Device: 186.85 Function: 86 S=85
RANDOM: NEC2 Device: 186.85 Function: 92 S=85
NEXT DISC: NEC2 Device: 186.85 Function: 93 S=85
MUTE ALL: NEC2 Device: 186.85 Function: 223 S=85
```

### Analyzing these signals on the air

Analyzing the signal timing on the original remote is possible using a Saleae Logic Analyzer as shown above. But once I had built my own sender, I wanted to measure the signal on the air. In hindsight, it would not have been necessary to open the original remote, but I was curious what was inside anyway.

Analyzing the signal timing on the air is possible with an [RTL SDR dongle](http://www.rtl-sdr.com/buy-rtl-sdr-dvb-t-dongles/), [gqrx](http://gqrx.dk), and Audacity.

You can verify that something is sent on the frequency using the waterfall chart in [gqrx](http://gqrx.dk). I used the [Gqrx-2.6-1.glibc2.17-x86_64.AppImage](https://github.com/csete/gqrx/releases/download/v2.6/Gqrx-2.6-1.glibc2.17-x86_64.AppImage) on Linux. Because the application is provided by its author as an [AppImage](http://appimage.org), I just had to download, `chmod a+x ./Gqrx*` and run.

![gqrx_wf_20161127_153832](https://cloud.githubusercontent.com/assets/2480569/20649713/dc386ea6-b4c5-11e6-8e54-725710d34a90.png)

But gqrx is not suitable to really show the details of the protocol. So using gqrx, I converted the signals to audio. I played around with the settings until I heared clear "blip, blip" signals when I pressed keys on the remote. Then I recorded the audio. Then opened the audio in [Audacity](http://www.audacityteam.org/download/) which is also available for Linux as a convenient [AppImage](http://appimage.org). Looks like this:

<img width="784" alt="Audacity" src="https://cloud.githubusercontent.com/assets/2480569/21288297/9d4aeed4-c47f-11e6-9379-1da1f1a1242b.png">

Bingo, the signal!

## Generating these signals with Raspberry Pi

__Known to work!__

How could I possibly generate these signals with only household items that I had already lying around?

[Here](https://discuss.ninjablocks.com/t/rtl-sdr-and-usb-receivers-for-433mhz-devices-and-lots-of-other-frequencies/1426/8) user joyceda suggested back in 2013:

> I was wondering if this PiFm 'hack' http://www.icrobotics.co.uk/wiki/index.php/Turning_the_Raspberry_Pi_Into_an_FM_Transmitter38 , an rtl-sdr and a 20 cm piece of wire would help to provide a generic RC tranciever unit for a ninjablock. At present it seems that this is limited 1Mhz up to 250Mhz - so my Bose is in

So I gave it a try, and what should I say: __It works__ using https://github.com/bskari/pi-rc and a long (ca. 50 cm - not yet optimized!) antenna. (In the beginning I tried with a 15 cm antenna and it would only work when I would touch the antenna.)

```
sudo ./pi_pcm -v
cat mute.json > /dev/udp/127.0.0.1/12345
```

This is the contents of mute.json, which is a representation of the MUTE timings `+9024 -4512 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -1692 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -564 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -1692 +564 -38628`:

```
[
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 9024,
        "spacing_us": 4512,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 3
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 2
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 1
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 564,
        "repeats": 8
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692,
        "repeats": 7
    },
    {
        "frequency": 27.145,
        "dead_frequency": 49.830,
        "burst_us": 564,
        "spacing_us": 1692000, 
        "repeats": 1
    }
]
```

To make things simpler, I wrote the `bosecontrol.c` code below which __works for me__ when the Raspberry Pi 1 is running at its normal 700 MHz frequency. Underclocking it results in the code not working. Using something like DMA rather than the CPU to do the switching would be much better (can you do it?).

### Ideas for further investigation

Being able to control a 27.145 MHz BOSE Lifestyle system with a Raspberry Pi may be a great proof-of-concept, but is not very practical for productive use (i.e., to be hidden permanently beyond the audio system as an additional way to control it):

 * The Raspberry Pi is overkill
 * The Raspberry Pi takes too long to boot
 * The Raspberry Pi consumes too much power for "just a remote control receiver"
 * The Raspberry Pi is large
 * The Raspberry Pi doesn't have WLAN built in
 
Hence, I am looking for a different solution.

## Generating these signals with Arudino

Might be possible with a __AD9850__ module
http://blog.riyas.org/2014/06/computer-controlling-27mhz-remote-control-car-ad9850-dds.html

Or with a __Si4012__. The Si4012 is a fully integrated FSK/OOK crystal-less CMOS high-data rate RF transmitter designed for the sub-GHz ISM band. 
http://www.silabs.com/products/wireless/EZRadio/Pages/si4012.aspx

Or with a __LTC6903__ Port Programmable Oscillator which can do 1 kHz to 68 MHz. 

Or with a __crystal__. Similar to [this circuit](http://www.pyroelectro.com/projects/pyro_rf_transmitter_27mhz/index.html) but using an Arduino instead of 5N555.

According to the ![Schematic](http://www.pyroelectro.com/projects/pyro_rf_transmitter_27mhz/img/output_power.png), the minimal BOM needed seems to be

* Need PCB, no Breadboard (since RF)
* 27.145 MHz Crystal
* 2x 2N2222 Transistors
* 100kΩ Resistor
* 100Ω Resistor
* 27pF Capacitor
* 68pF Capacitor
* 100pF Capacitor
* 150pF Capacitor
* 3x 2.2uH Inductor Choke

I have not tried this yet due to the lack of hardware. Please let me know if you do.

RC Toys seem to use the TX-2B chip (which we could replace with an Arudino) and this more elaborate circuit:

<img width="661" alt="circuit" src="https://cloud.githubusercontent.com/assets/2480569/21289502/1db558ce-c49f-11e6-98c9-74597a4b4cb5.png">

Seems to be relatively [complex](http://www.edaboard.com/thread307034.html). Coils? Inductors? Is that really all needed? probably the extra component are for filtering, which may be advisable.

I searched eBay for ready-to-use modules, but unlike for 433 MHz, readymade modules for this frequency seem not to exist. Why is this?

## Generating these signals with ESP8266

__It works__ with something along the lines of https://github.com/cnlohr/channel3, with [esp8266/Arduino](https://github.com/esp8266/Arduino) that lets us use the familiar Arduino toolset. Sketch below.

Someone is doing something very similar but for 433 MHz at https://github.com/papadeltasierra/dma433

However, it generates many other frequencies as well, so this is not a clean solution. The sender with a 30 cm long wire as an antenna on pin RX needs to be placed not too far from the music center.

## Other investigations into the system

### Teardown of the Control unit

The bose __Lifestyle 8 Series II__ (2001) system consists of
* "Lifestyle® music center" (control unit with built-in tuner and CD player)
* "Acoustimass module" (subwoofer with integrated amplifier and bass/treble control dials)
* 5 cube Speakers

The music center has as its main ICs:

* __SANYO LA1836__ Single-Chip Home Stereo Electronic Tuning IC
* __SANYO LA6541__ 4-channel Bridge Driver for Compact Discs
* __SANYO LA9241M__ Analog Signal Processor (ASP) for CD players
* __SANYO LC78622E__ Compact Disc Player DSP ("realizes an optimal cost-performance tradeoff for low-end players by strictly limiting functionality to basic signal-processing and servo system functionality") - BOSE = low-end?!?
* __SANYO 72131__ AM/FM PLL Frequency Synthesizer
* __ST TDA7309__ Digital Controlled Stereo Audio Processor with Loudness. "The TDA7309 is a control processor with independent left and right volume control for quality audio applications. Selectable external loudness and soft mute functions are provided. Control is accomplished by serial I2C bus microprocessor interface."
* __ST TDA7310__ Serial Bus Controlled Audio Processor "The TDA7310 is a volume, tone (bass and treble) and fader (front/rear) processor for high quality audio applications in car radio and Hi-Fi systems. Loudness and selectable input gain are provided. The control of all fuctions is accomplished by serial bus microprocessor interface."
* __ALPS 191401-001__ Tuner?

I did not find a general purpose MCU but I did not take the boards out, possibly it is on the other side of the PCB.

Where is the BOSE secret sauce hiding?

### Running the Acoustimass module without the music center

Officially the music center is needed to run the Acoustimass module and its connected speakers. The music center is used to switch on/off the device and set the volume, balance etc. But the input to the Acoustimass module consists of a proprietary "Audio input" connector that has on the other end

* Cinch stereo cable (regular analog signal, presumably)
* 3-pin "System control" (this we could possibly emulate using an Arduino if we knew the protocol - please let me know if you do, e.g., measure it with an oscilloscope and if signals are below 5V check with a Logic Analyzer; is it similar to US Patent 2005/0289224 A1?) (3.5 mm headphones-like connector)
* Digital input cinch female input

According to [this thread](http://www.ecoustics.com/electronics/forum/home-audio/545019.html), the pinout is like this. (I still need to verify that thiss really applies for the Lifestyle 8 Series II).

Here is the Acoustimass module side:

![Pinout](https://cloud.githubusercontent.com/assets/2480569/21572405/dbf18478-ced8-11e6-9ec8-199eccb8d46c.jpg)

On the music center side:

> You will need a pair of shielded phono cables for the left and right audio inputs and a stereo 3.5mm phone plug and 2 conductor shielded cable for the control/data cable. The tip of the phone plug will connect to pin 12 on the AM800P din plug, the ring of the phone plug to pin 7, and the shaft (shield) connects to pin 3. For the audio cables, left signal to pin 9, right signal to pin 1 and both shields to pin 5. Also jumper pins 2,3,6 and 10 together. They are all ground but do not connect them to pin 5, leave that connected to the signal ground only.

> I might also mention here that the 10-12V turn-on voltage is NOT an absolute necessity. The AM5P has two power supplies and one is always on when the sub is connected to AC.(Unless it's a 220/240V version which has a power switch that must be on also.) You can either power the sub on using an applied voltage (10-12VDC) to pin 1 of the DIN connector or simply rely on the "Audio Sense" circuitry to power the sub on in the presence of an audio signal. The sub will power down to standby mode after about two minutes of a "no signal" condition.

With "phone plug" I assume they mean a 3.5 mm "headphone" jack.

### Controlling the music center with a infrared remote control

Now that we know that the music center operates with NEC2 codes but uses RF rather than IR to submit them, I am curious whether we could hook up an infrared receiver (e.g., Vishay TSOPxxxx) instead of/in addition to the RF receiver built into the music center. However, I was not able to identify the RF receiver inside the music center but I did not take the boards out, possibly it is on the other side of the PCB.

What I DID find, to my surprise, was an unpopulated 3-pin connector __J400__ close to the display, and, even more interestingly, mounting holes for a 3-pin device right __Q400__ in front of the PCB where the locical location for an infrared receiver would be, with silkscreen suggesting a device looking very much like an infrared receiver (e.g., Vishay TSOPxxxx). When I place one there, it matches up _perfectly_. Unfortunately the traces are on the other side of the PCB so I could not trace them. I did not take the boards out. Could it be that BOSE engineers designed the PCB for an optional infrared receiver which then was left out in the end to save a few cents or make for a cleaner front housing design? Can we put one in there?

### Providing digital sound
The manual says: "If your Lifestyle® system receives a valid digital signal (including PCM or Dolby Digital bitstreams), this digital sound is used." 

Questions:
* How do I provide this from a ALSA USB sound card? To be investigated. 
* Do I need a USB sound card with a SPDIF output? 
* How do I need to configure ALSA?
* Is "Dolby Digital" == "AC3"?

Please let me know if you know.

Looks like I need IEC958: "To enable SPDIF output, you need to turn on “IEC958 Output Switch” control via mixer or alsactl (“IEC958” is the official name of so-called S/PDIF)." https://01.org/linuxgraphics/gfx-docs/drm/sound/cards/cmipci.html

A "HDMI Audio Extractor" may be able to extract the required signal from a Raspberry Pi... are there easier ways?

A "PCM2704 USB DAC to S/PDIF Sound Card Decoder Board" from China for under 5 USD may do it too. I ordered one for 3.58 USD called "5V USB Powered PCM2704 MINI USB Sound Card DAC decoder board for PC Computer T9". I will have to solder to pin 5 of the IC. Didn't like the bulkier (and more expensive) ones which require a old-school USB cable.
