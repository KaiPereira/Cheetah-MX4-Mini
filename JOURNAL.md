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

## Day 2 - Finishing MCU Schematic - 10 Hours

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

The MCU is looking pretty sick now! Time to implement CAN in! But while researching whether to put CAN in or now, I found out that the TMC2209 doesn't actually support it, and instead I need UART. So I actually did finish implementing it, but now I just need to use UART instead so I'm not going to bother.

Something I neglected though, was adding in the PSU input for the board, so let's add that real quick. You'll notice the board is constantly being changed with some new formatting, just because everything is getting bigger too.

And now we have the PSU in the top corner.

![[Pasted image 20250722165346.png]]
It might be a bit wrong, but I'll figure that out in a bit, for now, we need to convert the 12/24V down to 3.3V for the MCU.

It's a bit complicate to wire, but basically just steps down to 3V3, and then stabilizes and protects it using some capacitors and diodes, and then it needs some internal voltage too, which is what the FB pin is for.

![[Pasted image 20250722165430.png]]

And just like that, we have a buck converter for the PSU! But I still kind of need to convert the 5V from the USB down to 3.3V for the MCU, so I might need to re-use the current buck converter if I can, or add another one...

But anyways, after reaching out to some people, they suggest I use a different buck converter because of the bad switching frequency, and also convert to 5V instead of 3.3V and then use an LDO to get from a 5V rail to 3.3V. So after a bit research, I'm just going to use the TPS54331 chip.

So after an absurdly long time, I got the buck converter wired for 12/24V to 5V.

![[Pasted image 20250723124747.png]]

I know I don't journal everything I do with this schematic because there's so much stuff, but I'm constantly making iterations, so I also replaced all the new labels with power labels and re-organized some stuff.

Anyways, that's the majority of the MCU stuff wired now. I spent a really long time working on this stuff today, and I got a lot done!

## Day 3 - Power and Drivers - 10 Hours

Today was all about finishing the power schematic, and adding in the stepper drivers and maybe some other things into the schematic.

So first things first, I need to have a common 5V rail. You see, both the USB and the PSU with the buck converter, are going to output 5V, but they can't both output them at once, so I need to output only one of them if they're both on. I'm going to prioritize the PSU, because the PSU also provides a 12/24V power, so it's going to be preferential if they're both connected.

I'll use a simple P-Mosfet circuit:

![[Pasted image 20250723171957.png]]

And then I need to add a linear regulator/LDO to step down the 5V to 3.3V for the MCU and some other operations. You see, I need the 12/24V for the steppers, the 5V rail for some fan gate and other operations, and then the 3.3V for the MCU and stuff, so it's important that I have all of these rails separate. I'm going to use the same LDO as the Manta M4P, the AMS1117 and then of course decouple it to stabilize the voltage.

![[Pasted image 20250723172121.png]]

Now... we can finally get to working on the stepper drivers! The MCU and power is finished so let's add the X driver, I'm going with stepstick, because I like how easy it is to assemble and it has a bunch of extra features which are nice, and it looks sick on boards instead of using an onboard chip.

So let's add the first motor driver... it's a bit tough to wire, but I think this should be good. I'm using UART to control everything and then I expose it to a 4 pin connector for the steppers.

![[Pasted image 20250723205431.png]]

Now I'm going to add the endstop pins... I want to be able to switch between sensorless and endstop homing, so I'm going to include both of them, just as separate GPIO's. The Manta M4P does a jumper switch approach instead, but I'm not low on GPIO's, so I feel like controlling this in firmware is more convenient.

While adding the endstops, I also fixed the wiring for the TMC2209 silent step stick, because I was basically wiring it for the IC instead of the module, and I also added caps and pullups/pulldowns and whatnot.

![[Pasted image 20250724002036.png]]

And then I kind of forgot to add decoupling caps on the power pins so here:

![[Pasted image 20250724005344.png]]

And then after that, I looked into it a bit more, and I messed up the single line UART, you need to converge both the RX and TX UART pins to one line, but I was just using one IO.

![[Pasted image 20250724174813.png]]

After that, I planned out all the UART pins on the MCU so I didn't use them up accidently.

Now, I'm like 85% confident that the schematic for the motor driver is functional, so I'm confident enough to clean up the schematic, and add all 4 drivers in (x, y, z, e0).

![[Pasted image 20250724181300.png]]

Now the schematic is looking INSANELY cool.

![[Pasted image 20250724181354.png]]

Now this day actually took me a really long time, because the motor drivers kept on having to be re-iterated on, and the parts were pretty complicated. At least I finally have the main part of the board on, so I can kind of get to adding features I want.

## Day 4 - Funtime :D

Today was all about adding cool features I want on the board like fans, thermistors, etc.

All the wiring should be easier from here on out happily, but I still need to be wary about protection and stuff.

So first thing I'm going to add to the board, is the thermistors. I've decided on going with 3 thermistors because if people want more, they can always add an I2C bus, but most people aren't using more than 3. The connector is just GND with an IO pin, with some pullups and a protection for transient voltage spikes/surges.

![[Pasted image 20250724215625.png]]

Now I can just multiply this for the 3 thermistors I want:

![[Pasted image 20250725000144.png]]

Next, I'm going to add a connector for a servo port, because people sometimes like to have a servo for things like a bed leveler or a filament changer.

![[Pasted image 20250725003545.png]]

And then I can do this cool thing, where I add an inductive probe connector, but this also functions as a BLTouch port using both the servo port AND this inductive probe connector, so it's like a triple function.

![[Pasted image 20250725010157.png]]
