# How to Write Gecko Codes

### What This Guide Covers
- How to write gecko codes in the year 2024. 
- Some tips on using the dolphin debugger.
- Bundling multiple Gecko codes.

### What This Guide Does NOT Cover
- AR Codes.
- Obscure and lesser used gecko code commands.
- PowerPC assembly. Here is a [WiiBrew Guide](https://wiibrew.org/wiki/Assembler_Tutorial), and [two](https://web.archive.org/web/20160307100538/https://www.nxp.com/files/product/doc/MPCFPE32B.pdf) [manuals](https://math-atlas.sourceforge.net/devel/assembly/ppc_isa.pdf).
- What a debugger is.
- What hex/hexadecimal is.

This guide is meant for people who are comfortable programming at a low level.

## What is a Gecko Code?
A gecko code provides a straightforward method of modifiying a game without needing to modify the actual iso file.

There are two main commands used in gecko codes:
- `04`: Overwrite one instruction in the code.
- `C2`: Insert any amount of instructions in the code.

[There are more commands](https://web.archive.org/web/20191001120524/https://www.geckocodes.org/index.php?arsenal=1),
but I haven't seen them used so I won't be covering them.

These commands modify *code* at boot time, not the iso. **The code is a very small portion of an iso**.
This code is contained in the DOL, [a special file in the iso](https://wiibrew.org/wiki/DOL).
The DOL is loaded into a special offset in RAM when the game boots up.
For Melee, DOL offset is 0x8000_0000.

If you want to modify a character's jump height using these commands, you can't find the *jump_height* location in RAM, 
then use a gecko command to modify that address.
Instead, you need to modify the game's *code* by insterting your own *code* that sets the jump height.
You can only write code.

I should mention there is some amount of static, inline data in the DOL.
Some lookup tables and constants are located there.
For example, function pointer tables for action states.


## Anatomy of a Gecko Code

Here is a gecko code:
```
$Flatzone Everywhere [Aitch, Others]
C2008404 00000003 # zero z axis in Fighter_UpdateHurtboxPosition
38000000 901f0030
901f003c 881f0024
60000000 00000000
C207ae6c 00000002 # zero z axis in Hitbox_UpdateHitboxPositions (previous frame hitbox position)
38000000 901f0054
8001002c 00000000
C207ae04 00000003 # zero z axis in Hitbox_UpdateHitboxPositions (newly instantiated hitboxes)
38000000 901f0054
901f0060 881f0043
60000000 00000000
0407add4 901f0054 # zero z axis in Hitbox_UpdateHitboxPositions (new frame hitbox position)
0407add8 901f0060 # zero z axis in Hitbox_UpdateHitboxPositions (new frame hitbox position)
*Removes Z-axis shenanagins from player hitboxes and hurtboxes.
```

- The Gecko Code title starts with a `$` character. The authors are in square brackets. There can only be one of these lines.
- Additional comments start with `*`. There can be any amount of these. Put these before or after the bulk of the code.
- Code comments start with `#`. These are not shown to the user. These are for the programmers.
- There must always be two sets of 8 hex digits on each line in the code.
- There must not be empty lines.

## Commands

### The `04` Command
This command replaces one asm instruction in the DOL.
Here is an example 04 command:
```
0407add8 901f0060
```

This command is always one line. Here are the parts:
```
This is the command   This is the instruction to inject
 v                          v
04     07add4             901f0054
          ^
 This is the offset into the DOL to inject
```

Notice that the DOL offset is only 24 bits.
This is an offset into the DOL, not a RAM address.

**This instruction won't be injected at address 0x0007ADD4**.
It will be injected into the DOL offset + this offset.
For Melee, the injected RAM address is 0x8007ADD4.

### The `C2` Command
This command injects multiple asm instruction into the DOL.
It is more powerful, but more complicated.

Here is a C2 command:
```
C2008404 00000003
38000000 901f0030
901f003c 881f0024
60000000 00000000
```

The first line is special.
```
This is the command   This is how many LINES of instructions to inject
 v                          v
C2   008404             00000003
        ^
 This is the offset into the DOL to inject code
```
Please note that this lists the number of *lines of instructions*, **NOT** the number of instructions to inject.
This has tripped me up.
In this command, we inject 5 asm instruction in 3 lines.

These are the injected instructions:
```
38000000 # li r0, 0
901f0030 # stw r0, 0x30(r31)
901f003c # stw r0, 0x3c(r31)
881f0024 # lbz r0, 0x24(r31)
60000000 # nop
```

Injection is implemented with a branch to your injected code, then a branch back.
These branches will be automatically inserted.

In the above example, I only intended to inject the first three instructions.
`881f0024 # lbz r0, 0x24(r31)` is the instruction at the injection offset.
You need to append this instruction to the injection, otherwise it will be overwritten with the branch instruction.

The final nop is added for padding.
The `C2` command must be terminated with zeroes at the end of a line.

## The Actual Process Of Writing A Gecko Code
### Setup
Make sure you have the latest version of dolphin. 
The debugger has gotten exponentially better lately.
Open dolphin with the `-d` flag. This opens the debugger.

Then open the game. Hit `Symbols->Load Map File`.
If you are creating a code for Melee, use `Symbols->Load Other Map File` and select GTME01.map from [here](https://github.com/AlexanderHarrison/TrainingMode-More).

### Tips and Addendum
- If you've used a graphical debugger before, dolphin should be fairly straightforward to navigate.
It can be prone to crashing, so make savestates.
This won't restore breakpoints unfortunately.

- I use dolphin's replace instruction feature to test out changes.
Loading a savestate will restore any instructions replaced after the savestate.
It will not remove breakpoints.

- To convert human readable asm to hex, use [this site](https://disasm.pro/).
Ensure it is set to PowerPC and Big Endian.
Note that **registers are not prefixed with 'r' on this site**.
For more complex codes use [this tool](https://github.com/JLaferri/gecko).

- Make notes of addresses, offsets, etc.
There is a LOT to keep track of, you don't want to keep pointer chasing.
If something is heap allocated, use savestates to avoid needing to repeatedly find addresses.

- If you want something to stop occuring, try to find the function that does it,
then prevent it from being called. You could replace the calling instruction with a nop,
or let the function occur then overwrite the result.

- If you want to add functionality, try to abuse debug features of your game.
Lots of games have debug functions to display text on the screen, change model colours, etc.

- [This site](https://celestialamber.github.io/rlwinm-clrlwi-decoder/) decodes the rlwinm and clrlwi instructions to C code. 
This makes these otherwise impenetrable instructions readable.

- Technically, the offset into the DOL is 25 bits, not 24.
If you need to set the 25th bit, you write the commands `C3` or `05`.

- This injection pattern lets you save state between injection invocations.
It also allows you to give labels to the state locations.
This is supported by gecko, but may not be supported by disasm.pro (at least in this format):
```
.macro tableLoadW regTarget, regTable, table, label

lwz \regTarget,  \label - \table(\regTable)
.endm

.macro tableSaveW regSource, regTable, table, label

lwz \regSource,  \label - \table(\regTable)
.endm

.macro tableLoadPtr regTarget, regTable, table, label

addi \regTarget,  \regTable, \label - \table
.endm


bl Code

###### Link table ######
Table: 

WordConst1: .long 0
Text1: .string "text1"
.align 2

###### Link table end ######

Code:

mflr r12   #load link table pointer to r12
tableLoadW r3, r12, Table, WordConst1   # load word constant 'WordConst1' to r3
tableLoadPtr r4, r12, Table, Text1    # load pointer to 'Text1' string variable to r4
tableSaveW r3, r12, Table, WordConst1   # save r3 to word constant 'WordConst1'

```

## Bundling Gecko Codes
I've never used it, but [gecko](https://github.com/JLaferri/gecko) can help you merge multiple codes.
This assembles multiple codes into one larger code, making iteration faster and easier.

It also includes the powerpc-eabi-as.exe exe which you can use to assemble ppc if you need a custom scripted workflow.
This binary is simply taked from the DevKitPro installation.
You can get a powerpc-eabi-as binary for linux as well by downloading DevKitPro.
