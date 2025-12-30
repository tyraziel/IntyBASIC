# IntyBASIC Quick Reference

**For Intellivision game development. Keep this file in your project root for Claude Code to reference.**

## Hardware Constraints

- **CPU:** CP1610 @ ~894 kHz
- **MOBs (sprites):** 8 total (STIC chip)
- **BACKTAB (screen):** 20×12 tiles (240 positions, indexed 0-239)
- **GRAM:** 64 cards (256-319), runtime definable
- **GROM:** 256 cards (0-255), ROM-based
- **Frame budget:** 16.67ms @ 60Hz (NTSC), 20ms @ 50Hz (PAL)
- **Variables:** ~228 8-bit, ~47 16-bit (or 7962 with --jlp)

## Quick Syntax Reference

### Variables
```basic
x = 5              ' 8-bit variable
#score = 1000      ' 16-bit variable (# prefix required everywhere)
DIM enemies(10)    ' Array, 0-indexed (11 elements: 0-10)
CONST MAX = 8      ' Constant
```

### Graphics
```basic
' Define graphics
ship_gfx:
BITMAP "XXXXXXXX"
BITMAP "X......X"

' Load to GRAM (max ~18 cards/frame, less with PLAY)
DEFINE 0, 4, ship_gfx  ' Load 4 cards starting at card 0
WAIT                   ' Synchronize

' Screen output
CLS                              ' Clear screen
PRINT AT 0, "HELLO"              ' Print string at position
PRINT AT row * 20 + col, "\5"    ' Print card #5
PRINT AT 42, (5*8+6)             ' Direct poke: card 5, color 6
```

### MOBs (Sprites)
```basic
' SPRITE mob_num, x, y, card_and_color
SPRITE 0, ship_x + $300, ship_y + $200, CARD_SHIP + 6

' X coordinate bits:
'   0-7: position (0-168)
'   8: interaction (collision detection)
'   9: visibility
'   10: double width
' Common: +$300 = interaction + visibility

' Y coordinate bits:
'   0-6: position (0-95)
'   7: 16-line sprite (use even GRAM card: 0,2,4...)
'   8-9: scale (00=0.5x, 01=1x, 10=2x, 11=4x)
'   10: X flip
'   11: Y flip
' Common: +$200 = 1x scale

' card_and_color bits:
'   0-2: color low bits
'   3-11: card number (0-255 GROM, 256-319 GRAM)
'   12: color high bit
'   13: priority (0=over bg, 1=under bg)

' Disable MOB
SPRITE 7, 0, 0, 0
```

### Input
```basic
' Controller disc
IF CONT1.LEFT THEN x = x - 1
IF CONT1.RIGHT THEN x = x + 1
IF CONT1.UP THEN y = y - 1
IF CONT1.DOWN THEN y = y + 1

' Buttons
IF CONT1.BUTTON THEN    ' Any button
IF CONT1.B0 THEN        ' Top buttons
IF CONT1.B1 THEN        ' Bottom-left
IF CONT1.B2 THEN        ' Bottom-right

' Keypad (needs WAIT for debounce)
IF CONT1.KEY <> 12 THEN  ' 0-9, 10=clear, 11=enter, 12=none

' CONT = both controllers, CONT2 = right, CONT3/4 = ECS
```

### Control Flow
```basic
' Subroutines (must be PROCEDURE)
main_loop:
    WAIT
    GOSUB update
    GOTO main_loop

update: PROCEDURE
    ' code
END                ' END implies RETURN

' Loops
FOR i = 0 TO 9
    PRINT i
NEXT i

FOR i = 5 TO 1 STEP -2
NEXT i

WHILE expr
    ' code
WEND

DO WHILE expr      ' Also: DO UNTIL expr
    ' code
LOOP

EXIT FOR / EXIT WHILE / EXIT DO

' Conditionals
IF expr THEN statement
IF expr THEN
    statements
ELSEIF expr THEN
    statements
ELSE
    statements
END IF

' Multi-way
ON expr GOTO label1, label2
ON expr GOSUB proc1, proc2
```

### Collisions
```basic
' Hardware collision (after WAIT)
IF COL0 THEN  ' MOB 0 hit something
    ' bits 0-7: which MOB
    ' bit 8: background pixel
    ' bit 9: border
END IF

' Software collision
dx = ABS(ship_x - enemy_x)
dy = ABS(ship_y - enemy_y)
IF dx < 8 AND dy < 8 THEN
    GOSUB collision
END IF
```

### Sound
```basic
' PSG channels
SOUND 0, freq, vol    ' Channel A (0-4095 freq, 0-15 vol)
SOUND 1, freq, vol    ' Channel B
SOUND 2, freq, vol    ' Channel C
SOUND 3, freq, type   ' Envelope
SOUND 4, noise, mix   ' Noise

' Frequency = 3579545 / 32 / desired_hz  (NTSC)
' Frequency = 4000000 / 32 / desired_hz  (PAL)

' Music player
PLAY SIMPLE     ' 2 channels (SOUND 2 free)
PLAY FULL       ' 3 channels
PLAY NONE       ' Disable

music_data:
    DATA 8      ' Ticks per note
    MUSIC C4, E4, G4
    MUSIC S, S, S    ' S = sustain
    MUSIC -, -, -    ' - = silence
    MUSIC REPEAT
```

## Essential Patterns

### Dynamic MOB Assignment (Astrosmash technique)
```basic
' Assign MOBs to closest threats
mobs_used = 0
FOR i = 0 TO MAX_OBJECTS - 1
    IF object_active(i) THEN
        ' Manhattan distance (FAST)
        dx = object_x(i) - ship_x
        dy = object_y(i) - ship_y
        IF dx < 0 THEN dx = -dx
        IF dy < 0 THEN dy = -dy
        dist = dx + dy
        
        IF dist < THRESHOLD AND mobs_used < 7 THEN
            ' Promote to MOB
            SPRITE mobs_used + 1, object_x(i) + $300, \
                   object_y(i) + $200, CARD_ASTEROID + 4
            object_is_mob(i) = 1
            mobs_used = mobs_used + 1
        ELSE
            ' Draw as background tile
            screen_pos = (object_y(i) / 8) * 20 + (object_x(i) / 8)
            PRINT AT screen_pos, "\2"
            object_is_mob(i) = 0
        END IF
    END IF
NEXT i

' Disable unused MOBs
FOR i = mobs_used + 1 TO 7
    SPRITE i, 0, 0, 0
NEXT i
```

### Frame-based Animation
```basic
frame = frame + 1
IF (frame AND 3) = 0 THEN  ' Every 4 frames
    anim = (anim + 1) MOD 4
    SPRITE 0, x + $300, y + $200, (CARD_BASE + anim) + 6
END IF
```

### Spread Work Across Frames
```basic
' Don't do everything every frame
IF (frame MOD 4) = 0 THEN
    GOSUB expensive_task
END IF

IF (frame AND 7) = 0 THEN  ' Every 8 frames
    GOSUB update_distant_objects
END IF
```

## Optimization Rules

**ALWAYS:**
- Use `(frame AND 3)` instead of `(frame MOD 4)` - bitwise is faster
- Use Manhattan distance `ABS(dx) + ABS(dy)` not Pythagorean
- Multiply/divide by powers of 2 auto-optimizes to shifts
- Set interaction bit ($300 in X) for collision detection
- Minimize PRINT calls - they're expensive

**NEVER:**
- Use `SQR(dx*dx + dy*dy)` - extremely slow
- GOTO into/out of PROCEDUREs - flow control error
- Forget WAIT before checking COL0-7
- Load more than ~18 GRAM cards per frame (16 with PLAY, 13 with PLAY FULL)
- Use PEEK/POKE in tight loops

**STRATEGY:**
- Prioritize objects: what needs MOBs NOW vs what can be background
- Profile early: test on jzintv, watch for slowdown
- Think in frames: what updates every frame? every 4? every 8?

## Common Pitfalls

1. **Only 8 MOBs total** - use background tiles + dynamic assignment
2. **Arrays are 0-indexed** - `DIM A(10)` gives 11 elements (0-10)
3. **# prefix everywhere** - `x` and `#x` are DIFFERENT variables
4. **Screen is 0-239** - row * 20 + col, not (row, col)
5. **Coordinates differ** - MOBs use pixels, BACKTAB uses 8×8 tiles
6. **Collision needs interaction bit** - add $300 to X coordinate
7. **COL registers only valid after WAIT**
8. **GRAM limit** - only 64 cards (256-319)
9. **Keypad needs debounce** - wait for CONT.KEY = 12 before reading

## File Organization
```basic
' 1. Constants
CONST MAX_ENEMIES = 8
CONST CARD_SHIP = 1

' 2. Variables
DIM ship_x, ship_y
DIM #score

' 3. Graphics
ship_gfx:
    BITMAP "XXXXXXXX"
    BITMAP "X......X"

' 4. Init
    GOSUB init_game

' 5. Main loop
main_loop:
    WAIT
    GOSUB input
    GOSUB update
    GOSUB draw
    GOTO main_loop

' 6. Subroutines
input: PROCEDURE
    IF CONT1.LEFT THEN ship_x = ship_x - 1
END

' 7. Data
level_data:
    DATA 1,2,3,4,5
```

### Scrolling

```basic
SCROLL offset_x, offset_y, direction

' Parameters:
'   offset_x, offset_y: 0-7 pixels
'   direction: 1=left, 2=right, 3=up, 4=down (when crossing boundary)

' Smooth scroll right
scroll_x = scroll_x + 1
IF scroll_x >= 8 THEN
    scroll_x = 0
    GOSUB update_map  ' Load new column
END IF
SCROLL scroll_x, 0, 0

' CRITICAL: Adjust sprite positions!
' Scroll affects BACKTAB, NOT MOBs
screen_x = (world_x - camera_x) AND 255
SPRITE 0, screen_x + $300, screen_y + $200, card + color
```

**Note:** Using SCROLL reduces available 16-bit variables by 3

**Display:** 20×12 tiles (160×96 pixels), visible ~159×88-96 pixels

## Build Commands
```bash
# Compile
intybasic game.bas game.asm

# Assemble
as1600 -o game.bin -l game.lst game.asm

# Run
jzintv game.bin                 # Normal
jzintv --jlp game.bin           # JLP support
jzintv -v1 game.bin             # Intellivoice

# Debug
as1600 -j game.smap -s game.sym -o game.bin -l game.lst game.asm
intysmap game.smap
jzintv -d game.bin --src-map=game.smap --sym-file=game.sym
```

## Useful Expressions
```basic
FRAME               ' Current frame number (0-65535)
NTSC                ' 1 if NTSC, 0 if PAL
RAND                ' Pseudo-random 0-255
RAND(n)             ' Pseudo-random 0 to n-1
RANDOM(n)           ' Force new random 0 to n-1
ABS(x)              ' Absolute value
SGN(x)              ' Sign: -1, 0, or 1
PEEK(addr)          ' Read memory
LEN(string)         ' String length
POS(expr)           ' Current screen position
VARPTR var          ' Pointer to variable/array
#MOBSHADOW(x)       ' Access MOB buffer (0-23)
#BACKTAB(x)         ' Access screen buffer (0-239)
```

## Memory & Advanced

### Fixed-point (8.8 format)
```basic
#pos = 10.5         ' Stored as $0A80
#pos = #pos +. 0.25 ' Fixed add
y = #pos AND 255    ' Extract integer part
```

### Assembly integration
```basic
result = USR my_func(arg1, arg2)  ' Call with return
CALL my_func(arg1, arg2)          ' Call without return
ASM INCLUDE "mycode.asm"          ' Include asm
```

### Memory maps (OPTION MAP)
```basic
OPTION MAP 1        ' 16K static (original Mattel)
OPTION MAP 2        ' 42K static (JLP/Atariage PCB)
OPTION MAP 3-7      ' Various bank-switching configs
```

## CP1610 Mindset

1. **Start with constraints** - 8 MOBs, 64 GRAM, variable counts
2. **Profile early** - test on jzintv, watch for slowdown
3. **Think in frames** - what updates when?
4. **Prioritize** - what needs hardware NOW?
5. **Keep it simple** - constraints reward pragmatic code

---

**This reference is based on IntyBASIC v1.5.1 manual**  
Keep it in your project root for Claude Code to reference when writing IntyBASIC code.
