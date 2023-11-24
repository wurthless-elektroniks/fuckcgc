# fuckcgc: crack/patch for Chicago Gaming Company Williams/Bally pinball remakes to enable colors

This was a project from years ago. I thought, "well, surely the pinball community is full of geniuses who would have figured this out sooner". Evidently this was not the case.

## TL;DR

1. Grab disk image off Chicago Gaming's site
2. Unzip disk image
3. Apply the prebuilt .ppf file to that disk image (.ppf patchers are readily available online)
4. Write patched disk image to 8 GB microSD card
5. Put microSD card into your game
6. Run installer as normal
7. Reboot your game and hope nothing blows up

## Supported images

**All of these are untested.** I do not have $10,000 to buy a pinball machine, so I cannot test them. You run the risk of ruining your game forever if you install these. Also beware that it will void any warranty that's on the game. If there even *is* still warranty, because the modern day pinball enterprise is a scam. 

Images supported are:

* MonsterBash103Installer.img

**If you have problems:** The good news is that you can write the original image to the SD card and install the unmodified software if need be. The installer is just a fancy interface to dd.

## Why do this?

The CGC remakes are amateur hour. Not only do they not fix any issues that the original games had, but they come with awful support from the company and even worse hardware design. The playfield is controlled entirely by a PIC microcontroller which, if it dies, will render the entire game unusable. Replacements are available but they cost hundreds of dollars.

Which leads us to the topic of discussion. The "color upgrade kits" in circulation are nothing more than a playfield microcontroller replacement that tells the game that it's allowed to display in color, as well as a SD card. But how, exactly, do they work? Well, I got bored a few years ago and looked into it, and had a good laff or three while I analyzed the code.

The CGC remakes run on Beaglebone Black hardware, and apparently whoever built the disk images for the software updates had been using his Beaglebone as a SNES emulator because there are traces of SNES ROMs present. But what was more amazing was the fact that this guy didn't use a cross-compiler and instead built the emulator on the Beaglebone itself, then deleted the sourcecode files afterward, saying, "okay, we can say this thing is closed source!"

Too bad. Data remanence is a bitch. You're able to recover a good deal of the emulator's source code using nothing more than a hex editor, including the color overlay code. That will probably be the focus of another project down the line (i.e., using the remakes' color data to colorize the original Williams machines).

## How the protection works

Basically speaking, there is a file in the source called cp.c ("copy protection"), and it sends supposedly random variables to the PIC. If the PIC does not return the expected responses, the game forces the color display feature off. However, if the game gets the responses it wants, it increases a "confidence" counter. Once that counter goes above 0x384, it says, "yeah, that's a color PIC".

A function called color_is_on(), which is called on the regular, does three checks, in this order:

1. Is the color display explicitly disabled in software, or is something else telling us to disable the color mode?
2. Is the confidence counter greater than 0x384?
3. Did we successfully load the cgc.so file (which contains the encrypted color overlay data)? (This one's kind of important: failing to do this will crash the game.)

If all conditions are met, return 1 (yes; enable color). Otherwise, return 0 (no; keep color disabled).

The crack is therefore trivial: skip the first two checks. (The color overlay data is always loaded regardless of if the game has a color PIC or not.)

## The crack

** Working patches coming soon. These only disable condition 2. I don't know, maybe it will work on your machine, but I don't have confidence in it.**

One of the following find-and-replace jobs will do the trick:

* Change 07 4B 1B 69 B3 F5 61 7F 05 DD to 07 4B 1B 69 B3 F5 61 74 00 BF
* Change 09 4B DA 68 B2 F5 61 7F 05 DD to 09 4B DA 68 B2 F5 61 7F 00 BF
* If you want to be *very* aggressive: Change F5 61 7F 05 DD to F5 61 7F 00 BF

## How to crack

### Warning

This method can leak information on the dirty pirate who cracked the software. Always choose a creatively named directory outside of the /home directory. Stay safe, and happy cracking.

### The basic workflow

1. Mount partition 3 of the main .img, or find where emmc.img is
2. Mount Linux partition of emmc.img
3. Find emumm in that Linux partition
4. Crack it as described above
5. Run "sync" in terminal to flush changes
6. Reboot your system (forces system to unmount all partitions)
7. Congratulations, you have a patched .img

## Pinball sucks

This will hopefully be the first of several pinball projects I do. The pinball business is dominated by profiteers who charge obscene amounts of money for shit that barely works, and they get away with it because they're the only game in town. Stern are just as guilty of this bullshit too.

If CGC ask politely I will take this repository down, but, in all honesty, I hope they leave it alone. All of the games that I aim to crack are now out of production anyway and have been out of production for years. There's no sense charging $500 for pretty pictures.

## Special thanks

Bad assumptions were made by Steve Ellenoff

## License

Public domain
