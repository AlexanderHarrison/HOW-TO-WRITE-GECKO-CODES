# How to Write Gecko Codes

### What This Guide Covers
- How to write gecko codes in the year 2024. 
- Some tips on using the dolphin debugger.

### What This Guide Does NOT Cover
- AR Codes.
- Obscure and lesser used gecko code commands.
- PowerPC assembly. [WiiBrew Guide](https://wiibrew.org/wiki/Assembler_Tutorial).
- What a debugger is.
- What hexadecimal is.

This guide is meant for people who are comfortable programming at a low level.

## What is a Gecko Code?
A gecko code provides a straightforward method of modifiying a game without needing to modify the actual iso file.

There are two main commands used in gecko codes:
- `04`: Overwrite one instruction in the code.
- `2C`: Insert any amount of instructions in the code.

These commands modify *code* at boot time, not the iso. **The code is a very small portion of an iso**.
This code is contained in the DOL, [a special file in the iso](https://wiibrew.org/wiki/DOL).
The DOL is loaded into a special offset in RAM when the game boots up.
For Melee, DOL offset is 0x8000_0000.

If you want to modify a character's jump height using these commands, you can't find the *jump_height* location in RAM, 
then use a gecko command to modify that variable.
Instead, you need to modify the game's *code* by insterting your own *code* that sets the jump height.
You can only write code.

[There are more commands](https://web.archive.org/web/20191001120524/https://www.geckocodes.org/index.php?arsenal=1),
but I haven't seen them used so I won't be covering them.

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

- The title of the gecko code starts with a `$` character and lasts til the end of the line.
The authors are in square brackets at the end.
There can only be one of these lines.
- Additional comments start with `*`. 
There can be any amount of these lines. 
Put these comments before or after the bulk of the code.
- Code comments start with `#`. 
These are not shown to the user. 
These are for the programmers.
- There must always be two sets of 8 hex digits on each line in the code.
- There must not be empty lines.

## Commands

### The `04` Command
Herre is an 04 command:
```
0407add8 901f0060 # zero z axis in Hitbox_UpdateHitboxPositions (new frame hitbox position)
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
This command is more powerful, but more complicated and also a little weird.
This command can be arbitrarily long.

Here is a C2 command:
```
C2008404 00000003 # zero z axis in Fighter_UpdateHurtboxPosition
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
