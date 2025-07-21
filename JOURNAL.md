---
title: 3D Printer Driver Board
author: Kai Pereira
description: A modern driver board for a 3D printer
created_at: 2025-07-20
---
## Day 1 - This is a **bad** idea - July 20th

I've decided to build a complex 3D printer driver board, similar to the Manta M4P or the SKR Mini E3, but with some added features I have yet to decide.

I'm not too good at PCB work, I've only designed like keyboards and macropads and such, so I'm excited for a way harder PCB project!

The problem is, I have just about 10 days to complete this entire or I won't get the grant! So let's get right into building it.

The first thing I had to decide on was the MCU. I decided on using the STM32H743, it's an insanely powerful MCU that has many features I love:
- 2 MB Flash, 1 MB Ram (for firmware and G-Code Interpreter and stuff)
- Ethernet MAC (so I can network the printer)
- 480 MHz Processor
- Support for every communication protocol imaginable
- Good firmware support
- Blazingly fast and powerful

So let's just add in the MCU:

![[Pasted image 20250721015238.png]]
Then of course, I'm going to connect all the simple pins like VDD and GND:

![[Pasted image 20250721015402.png]]
Next, each VDD needs a decoupling capacitor, and then a larger one for VDDA and VSSA:

![[Pasted image 20250721015504.png]]
Keep in mind, while I'm going through these fast, lots of these decisions take me hours, because I've never done this before and I have to re-iterate.

Anyways, next I forgot, but I need caps on VCAP so that the internal voltage of the chip is alright.

![[Pasted image 20250721015649.png]]

And then next, I'll add resetting and booting, which took me a really long time to figure out, but with some friends help, I got it down pretty good.

![[Pasted image 20250721015754.png]]
And then after that, I need to add my main crystal in

