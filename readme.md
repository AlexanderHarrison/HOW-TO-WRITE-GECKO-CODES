# How to Write Gecko Codes

### What This Guide Covers
How to write gecko codes in the year 2024. 

### What This Guide Does NOT Cover
- AR Codes.
- Obscure and lesser used forms of gecko codes.

## What is a Gecko Code?
A gecko code provides a straightforward method of modifiying a game, without needing to modify the actual iso file.
Instead, it provides commands which modify the game's code when it boots up.

Got that? Gecko codes modify *code*, not the iso. **The code is a very small portion of an iso.**

If you want to modify a character's jump height, you can't find the *jump_height* location in RAM, 
then use a gecko code to modify that location.
Instead, you need to modify the game's *code* by insterting your own *code* that sets the jump height.
There is no gecko command to write to RAM.
You can only write code.

There are two commands used in gecko codes:
- `04`: Overwrite one instruction in the code.
- `2C`: Insert any amount of instructions in the code.

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

The title of the gecko code starts with a `$` character and lasts til the end of the line.
The authors are in square brackets at the end.
There can only be one of these lines.

Additional comments start with `*`. 
There can be any amount of these lines. 
Put these comments before or after the bulk of the code.

Code comments start with `#`. 
These are not shown to the user. 
These are for the programmers.

There are usually multiple gecko commands in a gecko code.
The commands in the above example are:
```
C2008404 00000003 # zero z axis in Fighter_UpdateHurtboxPosition
38000000 901f0030
901f003c 881f0024
60000000 00000000
```
```
C207ae6c 00000002 # zero z axis in Hitbox_UpdateHitboxPositions (previous frame hitbox position)
38000000 901f0054
8001002c 00000000
```
```
C207ae04 00000003 # zero z axis in Hitbox_UpdateHitboxPositions (newly instantiated hitboxes)
38000000 901f0054
901f0060 881f0043
60000000 00000000
```
```
0407add4 901f0054 # zero z axis in Hitbox_UpdateHitboxPositions (new frame hitbox position)
```
```
0407add8 901f0060 # zero z axis in Hitbox_UpdateHitboxPositions (new frame hitbox position)
```
