---
title: 3D Printer Driver Board
author: Kai Pereira
description: A modern driver board for a 3D printer
created_at: 2025-07-20
---
## Day 1 - This is a **bad** idea - July 20th - 6 Hours

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
And then after that, I need to added my main crystal in. I chose 25 MHz because it's better for USB and ethernet clock accuracy for some complex reason, but it does.

![[Pasted image 20250721020815.png]]

## Day 2 - Finishing MCU Schematic

I got quite a bit of work done on the MCU yesterday, but today, let's finish it off, and potentially we can get started on working on the motor drivers.

First of all, I added Ferrite beads to VDDA so it doesn't receive any noise from the standard power line:

![[Pasted image 20250721180109.png]]
After that, I implemented SWD header debugging so I could debug the programming and manually flash and stuff.

![[Pasted image 20250721180218.png]]
Now there's some more things I need on the MCU, but I want to work on the SD Card and USB peripherals. First I'm going to put the SD Card in, because that one made a bit more sense to me, everything is basically data or power/GND and then you just put pull-ups on data's and pull-downs on powers.

![[Pasted image 20250721185006.png]]

And now I just have to add the USB-C and CAN ports, I chose USB because it's pretty modern and there's also no harm in having CAN, especially because it's such a high quality board, I think enterprises will like it.

I'm also going to have it where when the USB-C is plugged in, it's going to power the board, but you can also power it with an external power supply, this is just really convenient for development.

![[Pasted image 20250721225653.png]]

The MCU is looking pretty sick now! Time to implement CAN in!
