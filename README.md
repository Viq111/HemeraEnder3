# Hemera on Ender 3

Hello! This repo should (hopefully) help you put an Hemera onto an Ender 3 Pro in the year 2021 in a breeze.
I mostly created this because I was following [Hemera to Ender 3 - Complete guide](https://www.youtube.com/watch?v=oY1F7fUBHrc) but some information was outdated; hopefully this will help one or two people in the future.

This is not meant to be a complete guide but a diff to the video [Hemera to Ender 3 - Complete guide](https://www.youtube.com/watch?v=oY1F7fUBHrc) so you should go watch it first and then come back here.

As a headsup, you can definitely do the installation in three steps: first the Hemera, second adding the part-cooling fan (I still had great success without a part cooling fan at first), third the BLTouch.

## (Better) Parts to print + additional hardware

Contrary to the video, I feel like the two part build is pretty heavy and make you loose a lot of space for no reason.
I actually used the part from [E3D hemera Mount and cooler for Cr-10 and Ender 3 by Tragical](https://www.thingiverse.com/thing:4061250) which are much better and do not make you loose any build volume.
I will provide a thingiverse link for a small remix that makes the BLTouch mount a bit better.

### What you need to print:
- `Mount_for_Hemera.stl` (I have a WIP remix to make the BLTouch mount nut slots a bit bigger so it's easy to use even with shrinking (at the end of the day, you don't need something snug))
- `Duct_v3.stl` + `Mount_for_5015_fan.stl`
- `Mount_for_Bltouch.stl` (I will completely change this one, it bumps into the Y axis motor without triggering the end switch - not great)

### Additional hardware:

**Fan mount:**
- 2x M4 hex nuts
- 2x M4x22mm screws (if you want to be safe and not do back&forth to the store, also get some M4x20mm & M4x25mm, you can always reuse in the future)

**BLTouch mount:**
- 4x M3 hex nuts
- 4x M3x10mm (to be confirmed, working on it)

## Vref calibration

I couldn't find easily the documentation online about what is Vref, Irms (sometimes just called RMS) so I'm doing a full section on it so it's clear for you.
Vref calibration is actually pretty simple and you don't have to be very precise! It basically just set the upper limit on what current you can send to your stepper motor without blowing it.

What you need first:
- Know the motor drivers you have on your board (for me with a Creality Silent Board 1.1.5, that was TMC2208)
- Know the "Rsense" value installed on your board. You can look this up online or directly on your board (the resistance close to where you plug the E stepper motor should have a number on it). Most of the time, it should be "100" which is a 0.100 Ohm resistance. For me on the Creality Silent Board 1.1.5, Rsense=150

You then can refer to the [Hemera documentation](https://e3d-online.zendesk.com/hc/en-us/articles/360016249057-Hemera-Current-Adjustment-PDF-) ([alternate link if down](https://github.com/Viq111/HemeraEnder3/files/7080832/Hemera.Current.Adjustments.pdf))

For me with TMC2208, it will tell you to use Irms (which is basically max current divided by square root of 2 with some formula)
What's really not clear here is the computation is actually just a cross product which tells you at 2.5V, your driver will deliver x amp.
Refer to the driver's documentation to try to find that value. For TMC2208, the documentation is [here](https://github.com/Viq111/HemeraEnder3/files/7080838/TMC220x_TMC2224_datasheet_Rev1.09.pdf). Page 49, you can see with Rsense=100, Vref=2.5V => Irms = 1.77A. With Rsense=150, Vref=2.5V => Irms = 1.28A.

The Hemera should be driven with a max Irms= 1.33A*sqtr(2) = 0.94A.
With Rsense=150, I want 0.94A and for Vref=2.5V,Irms=1.28A. With a cross product, that gives me Vref = Irms_want / Irms_at_2.5 * 2.5V = 1.83V.
That's the **maximum** value you can ever set your Vref at before you burn your stepper motor.
It is recommended to set a value at ~80-90% of that maximum value to be safe. I initially set it to the value in the video (0.5V) and that definitely didn't work so I would recommend not setting it to < 70% max value.
As you can see, Vref doesn't actually need to be accurate, you can set it anywhere between 70% & 90% of you max value. In my case (Creality Silent Board 1.1.5), I want to set it between 1.28V & 1.65V.

Remember that:
- If you set your Vref too high, you can break your stepper motor so always do the computation and that will give you the upper bound
- If you set your Vref too low, your motor won't move or "skip steps" (so basically not extrude correctly). Since I never had a stepper motor overhead, I set mine at 90% of max and it has been doing fine.

tl; dr; Don't change Vref except if you run into issues. The stock value works except if the new stepper is 2x more powerful than previously. The Hemera stepper motor should be more powerful than the stock one so no risk of damaging it.


## Marlin 2 config

### Bootloader

I don't remember too much this step so I won't go too much into details. If you have an arduino, there should be plenty of guides. If you use the Creality BLTouch kit, you can follow this [reddit thread](https://www.reddit.com/r/ender3/comments/cfmbdy/howto_installing_a_bootloader_to_an_ender_3_pro/).

Two things worth mentioning:
- The `DC, D6, FD and FF` values are **correct** (opposite of what's in the BLTouch kit guide), this is because you are flashing the **bootloader and not the firmware**.
- There is one step not mentioned is to select the correct microprocessor that's on your board

After you have succesfully flashed the bootloader, you'll be able to repeatedly flash the firmware with an attached Octoprint (or similar) 

### Config

1. Download the latest firmware from https://github.com/MarlinFirmware/Marlin/releases
1. Download the correct config from https://github.com/MarlinFirmware/Configurations and override the `Configuration.h` and `Configuration_adv.h` with files for your printer
1. You can modify the values as shown in this commit: https://github.com/Viq111/HemeraEnder3/commit/8336145 (make sure to set the correct driver for the board you are using)
1. Use VS Code & PlatformIO to build (don't use Arduino IDE, that has been deprecated and is hard to get it working now)

Now you should have a working Ender 3 with a Hemera hotend!

### Adding BLTouch

*Coming soon* I was satisfied with the given stl for the BLTouch mount so I'm currently reworking it
