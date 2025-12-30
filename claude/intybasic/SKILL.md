---
name: intybasic
description: IntyBASIC programming reference for Intellivision development. Use when writing, reviewing, or debugging IntyBASIC code for Intellivision games. Covers syntax, MOB/sprite management, GRAM/GROM graphics, CP1610 hardware constraints, optimization patterns, and classic game techniques like dynamic object promotion and frame budget management.
---

# IntyBASIC Programming Guide

IntyBASIC is a BASIC-like language for Intellivision game development that compiles to CP1610 assembly.

## Core Hardware Constraints

**CPU:** CP1610 at ~894 kHz (not CP1600!)

**MOBs (Moving OBjects):** 8 hardware sprites total (managed by STIC chip)
- 8×8 or 8×16 pixels (single/double height)
- Can be doubled horizontally (16px wide)
- Hardware collision detection
- Each MOB has: X/Y position, card number, color, size/flip flags

**BACKTAB (Background):** 20×12 grid of 8×8 tiles
- 240 tile positions total (0-239)
- References cards from GROM/GRAM

**Cards:** 8×8 pixel graphics data
- GROM: ROM-based, cards 0-255 (fixed at compile time)
- GRAM: 64 slots, cards 256-319 (can be redefined at runtime)
- Multiple MOBs/tiles can share same card

**Frame Budget:** 16.67ms per frame at 60 Hz (NTSC) or 20ms at 50 Hz (PAL)
- Exceeding budget = dropped frames/slowdown

**Variables:**
- 8-bit: ~228 available (reduces with features: -3 SCROLL, -6 keypad, -28 PLAY)
- 16-bit: ~47 available (or 7962 with --jlp/--cc3 flags)

## Essential Syntax

### Graphics Definition

```basic
REM BITMAP - define graphics in pairs (DECLE format)
ship_graphic:
BITMAP "XXXXXXXX"
BITMAP "X......X"
BITMAP "X......X"
BITMAP "XXXXXXXX"

REM Alternative: character representation
BITMAP "........"  
BITMAP "....XXXX"  ' Any char besides 0_. is treated as 1

REM Alternative: numeric DATA
DATA 255, 129, 129, 255
```

### MOB/Sprite Control

```basic
REM SPRITE mob_number, x, y, card_and_color
SPRITE 0, ship_x + $300, ship_y + $200, CARD_SHIP + 6

REM X bits: 0-7=pos, 8=interact, 9=visible, 10=double-width
REM Y bits: 0-6=pos, 7=16-line, 8-9=scale, 10=x-flip, 11=y-flip  
REM card_and_color: 0-2=color_low, 3-11=card, 12=color_high, 13=priority

REM Disable MOB
SPRITE 7, 0, 0, 0

REM Common additions:
REM $300 = interaction + visibility bits for X
REM $200 = 1x scale for Y
```

### Background Tiles

```basic
REM PRINT AT position writes to screen
PRINT AT 0, "HELLO"              ' Top-left (0-239)
PRINT AT 19, "B"                 ' Top-right
PRINT AT 220, "C"                ' Bottom-left
PRINT AT row * 20 + col, "\5"    ' Card #5 at row/col

REM Clear entire screen
CLS

REM Direct screen poke (12-bit value)
PRINT AT 42, (5*8+6)             ' Card 5, color 6
```

### Input Handling

```basic
REM Controller disc
IF CONT1.LEFT THEN x = x - 1
IF CONT1.RIGHT THEN x = x + 1
IF CONT1.UP THEN y = y - 1
IF CONT1.DOWN THEN y = y + 1

REM Buttons
IF CONT1.BUTTON THEN GOSUB fire   ' Any button
IF CONT1.B0 THEN                  ' Top buttons (left/right)
IF CONT1.B1 THEN                  ' Bottom-left
IF CONT1.B2 THEN                  ' Bottom-right

REM Keypad (requires WAIT for debounce)
IF CONT1.KEY <> 12 THEN           ' 0-9, 10=clear, 11=enter, 12=none

REM CONT (no number) checks BOTH controllers
REM CONT2 for right controller, CONT3/4 for ECS
```

### Control Flow

```basic
REM Subroutines - must be PROCEDURE
main_loop:
    WAIT
    GOSUB update_game
    GOTO main_loop

update_game: PROCEDURE
    REM ... game logic ...
END                    ' END implies RETURN

RETURN                 ' Early return from procedure

REM Loops
FOR i = 0 TO 9
    PRINT i
NEXT i

FOR i = 5 TO 1 STEP -2  ' Counts 5, 3, 1
NEXT i

WHILE expr
    REM ...
WEND

DO WHILE expr          ' Also: DO UNTIL expr
    REM ...
LOOP

EXIT FOR / EXIT WHILE / EXIT DO

REM Conditionals
IF expr THEN statement
IF expr THEN statement ELSE statement

IF expr THEN
    statements
ELSEIF expr THEN
    statements  
ELSE
    statements
END IF

REM Multi-way branches
ON expr GOTO label1, label2, label3
ON expr GOSUB proc1, proc2, proc3
```

### Variables and Arrays

```basic
REM Variables created by use (no declaration needed)
x = 5                  ' 8-bit variable
#score = 1000          ' 16-bit variable (note # prefix)

REM Explicit declaration
DIM x, y, #score

REM Arrays (0-indexed)
DIM enemies_x(10)      ' 11 elements: 0-10
DIM #big_numbers(5)    ' 16-bit array

enemies_x(0) = 50
#score = #big_numbers(i)

REM Constants
CONST MAX_ENEMIES = 8
CONST CARD_SHIP = 1
```

## Classic Techniques

### Dynamic MOB Assignment (Astrosmash Pattern)

```basic
REM Assign MOBs to closest objects
mobs_used = 0
FOR i = 0 TO MAX_OBJECTS - 1
    IF object_active(i) THEN
        REM Manhattan distance (fast!)
        dx = object_x(i) - ship_x
        dy = object_y(i) - ship_y
        IF dx < 0 THEN dx = -dx
        IF dy < 0 THEN dy = -dy
        dist = dx + dy
        
        IF dist < THRESHOLD AND mobs_used < 7 THEN
            REM Promote to MOB
            SPRITE mobs_used + 1, object_x(i) + $300, \
                   object_y(i) + $200, CARD_ASTEROID + 4
            object_is_mob(i) = 1
            mobs_used = mobs_used + 1
        ELSE
            REM Draw as background tile
            screen_pos = (object_y(i) / 8) * 20 + (object_x(i) / 8)
            PRINT AT screen_pos, "\2"
            object_is_mob(i) = 0
        END IF
    END IF
NEXT i

REM Disable unused MOBs
FOR i = mobs_used + 1 TO 7
    SPRITE i, 0, 0, 0
NEXT i
```

### Animation

```basic
REM Frame-based animation
frame = frame + 1
IF (frame AND 3) = 0 THEN  ' Every 4 frames
    anim = (anim + 1) MOD 4
    SPRITE 0, x + $300, y + $200, (CARD_BASE + anim) + 6
END IF
```

### Frame-Rate Management

```basic
REM Spread expensive work across frames
IF (frame MOD 4) = 0 THEN
    GOSUB expensive_collision_check
END IF

REM Update distant objects less frequently
IF (frame AND 7) = 0 THEN  ' Every 8 frames
    GOSUB update_background_objects
END IF
```

### Collision Detection

```basic
REM Hardware MOB collision (automatic by STIC)
WAIT  ' Collisions update after WAIT
IF COL0 THEN  ' MOB 0 hit something
    REM bits 0-7: which MOB hit
    REM bit 8: hit background pixel
    REM bit 9: hit border
    GOSUB handle_collision
END IF

REM Manual collision (software check)
dx = ship_x - asteroid_x
dy = ship_y - asteroid_y
IF dx < 0 THEN dx = -dx
IF dy < 0 THEN dy = -dy
IF dx < 8 AND dy < 8 THEN
    GOSUB collision
END IF
```

## Optimization Tips

**Bitwise operations faster than MOD:**
```basic
IF (frame AND 3) = 0 THEN     ' Fast: every 4 frames
IF (frame MOD 4) = 0 THEN     ' Slower
```

**Manhattan distance over Pythagorean:**
```basic
dist = ABS(dx) + ABS(dy)              ' Fast
REM NOT: dist = SQR(dx*dx + dy*dy)   ' Very slow
```

**Optimized multiplication/division:**
- By 2/4/8/16/32/64/128/256: auto-optimized to shifts
- By constants < 128: uses fast macro
- Variable × variable: uses intvnut's fast multiply (272 cycles)
- Use --jlp flag for hardware acceleration

**Minimize PRINT calls:**
```basic
REM Cache positions, only update changes
```

**GRAM management:**
```basic
REM Load GRAM only when needed (max ~18 cards/frame)
REM Reduced to 16 with PLAY, 13 with PLAY FULL

DEFINE 0, 4, sprite_graphics  ' Loads 4 cards starting at 0
WAIT  ' Synchronize with load
```

## Common Pitfalls

- **MOB limit:** Only 8 total. Use background + dynamic assignment
- **GRAM limit:** Only 64 cards (256-319). Reload strategically
- **Frame budget:** Profile! Spread work across frames
- **Coordinate mismatch:** MOBs use pixels, BACKTAB uses 8×8 tiles
- **Arrays 0-indexed:** `DIM A(10)` gives 11 elements (0-10)
- **# prefix consistency:** `x` and `#x` are different variables
- **PROCEDURE flow:** Don't GOTO into/out of PROCEDUREs
- **Interaction bit:** Set bit 8 of X ($300) for collision detection

## Sound and Music

```basic
REM PSG channels (AY-3-8914 chip)
SOUND 0, freq, vol    ' Channel A (0-4095 freq, 0-15 vol)
SOUND 1, freq, vol    ' Channel B
SOUND 2, freq, vol    ' Channel C
SOUND 3, freq, type   ' Envelope
SOUND 4, noise, mix   ' Noise channel

REM Frequency calculation
REM freq = 3579545 / 32 / desired_hz   (NTSC)
REM freq = 4000000 / 32 / desired_hz   (PAL)

REM Music player (integrated tracker)
PLAY SIMPLE           ' 2 channels (leaves SOUND 2 free)
PLAY FULL             ' 3 channels (no SOUND available)
PLAY NONE             ' Disable player

music_data:
    DATA 8            ' Ticks per note
    MUSIC C4, E4, G4
    MUSIC S, S, S     ' S = sustain
    MUSIC -, -, -     ' - = silence
    MUSIC REPEAT
```

## Scrolling

IntyBASIC provides the SCROLL statement for pixel-perfect scrolling using the STIC's hardware scroll registers.

### SCROLL Statement

```basic
SCROLL offset_x, offset_y, direction
```

**Parameters:**
- `offset_x`: Horizontal pixel offset (0-7)
- `offset_y`: Vertical pixel offset (0-7)  
- `direction`: What to do when crossing 8-pixel boundary
  - 1 = Scroll leftwards (shift BACKTAB right)
  - 2 = Scroll rightwards (shift BACKTAB left)
  - 3 = Scroll upwards (shift BACKTAB down)
  - 4 = Scroll downwards (shift BACKTAB up)

**How It Works:**

The STIC can offset the display by 0-7 pixels without changing BACKTAB. Once you cross an 8-pixel boundary (tile edge), you must shift all BACKTAB content and reset the offset.

### Smooth Scrolling Algorithm

**Horizontal scrolling example:**
```basic
REM Scroll right continuously
scroll_x = scroll_x + 1

IF scroll_x >= 8 THEN
    scroll_x = 0
    REM shift_backtab updates the entire screen
    REM this is where you'd copy new column from your map
END IF

SCROLL scroll_x, 0, 0  REM 0 = no auto-shift this frame
```

**Using auto-shift:**
```basic
REM IntyBASIC can auto-shift when you hit pixel 8
scroll_x = scroll_x + 1

IF scroll_x >= 8 THEN
    scroll_x = 0
    direction = 2  REM Shift BACKTAB left (scrolling right)
ELSE  
    direction = 0  REM No shift
END IF

SCROLL scroll_x, 0, direction
```

### CRITICAL: Sprite Position Correction

**Scrolling affects BACKTAB but NOT MOBs!** You must manually adjust sprite positions to compensate.

```basic
REM World coordinates (where object really is)
DIM #ship_world_x, #ship_world_y

REM Scroll offset (camera position)  
DIM #scroll_world_x, #scroll_world_y

REM Calculate screen position
ship_screen_x = (#ship_world_x - #scroll_world_x) AND 255
ship_screen_y = (#ship_world_y - #scroll_world_y) AND 255

REM Display sprite at corrected position
SPRITE 0, ship_screen_x + $300, ship_screen_y + $200, CARD_SHIP + 7
```

### Variable Usage Note

**Important:** Using SCROLL anywhere in your program automatically reduces available 16-bit variables by 3, as IntyBASIC uses them for buffering.

### Display Boundaries

**BACKTAB:** 20 columns × 12 rows = 160×96 pixels

**Visible Display:**
- Horizontal: ~159 pixels (essentially full 20 tiles)
- Vertical: ~88-96 pixels (11-12 tiles, varies by TV)

For scrolling maps larger than screen:
- Maintain full map in memory (or load sections)
- Display 20×12 window of map in BACKTAB
- Update edge tiles when crossing boundaries
- Track camera/scroll position in world coordinates

### Border Masking

Use BORDER statement to hide edges during scrolling transitions:

```basic
BORDER color, mask
REM mask: 0=none, 1=left column, 2=top row, 3=both
```

This prevents visible glitches when updating edge tiles.

### Complete Scrolling Example

```basic
REM Side-scrolling game setup
CONST MAP_WIDTH = 100   REM Map is 100 tiles wide
DIM #camera_x           REM Camera world position
DIM scroll_pixel_x      REM 0-7 pixel offset

main_loop:
    WAIT
    
    REM Move camera right
    #camera_x = #camera_x + 2
    
    REM Calculate pixel offset
    scroll_pixel_x = #camera_x AND 7
    
    REM Check if crossed tile boundary
    IF scroll_pixel_x = 0 OR scroll_pixel_x = 1 THEN
        GOSUB update_backtab_columns
    END IF
    
    REM Apply scroll
    SCROLL scroll_pixel_x, 0, 0
    
    REM Update sprites with corrected positions
    GOSUB update_sprites
    
    GOTO main_loop

update_sprites: PROCEDURE
    REM World position - camera = screen position
    ship_x = (#ship_world_x - #camera_x) AND 255
    SPRITE 0, ship_x + $300, ship_y + $200, CARD_SHIP + 7
END

update_backtab_columns: PROCEDURE
    REM Load new column of tiles from map into BACKTAB
    REM This is your map-to-screen transfer code
END
```

## File Organization Pattern

```basic
REM 1. Constants
CONST MAX_ENEMIES = 8
CONST CARD_SHIP = 1

REM 2. Variables
DIM ship_x, ship_y
DIM #score

REM 3. Graphics
ship_gfx:
    BITMAP "XXXXXXXX"
    BITMAP "X......X"

REM 4. Init
    GOSUB init_game
    GOSUB setup_graphics

REM 5. Main loop
main_loop:
    WAIT
    GOSUB input
    GOSUB update
    GOSUB draw
    GOTO main_loop

REM 6. Subroutines
input: PROCEDURE
    IF CONT1.LEFT THEN ship_x = ship_x - 1
END

REM 7. Data
level_data:
    DATA 1,2,3,4,5
```

## CP1610 Mindset Checklist

1. **Start with constraints:** MOB budget (8), GRAM budget (64), variables
2. **Profile early:** Test on jzintv, watch for slowdown
3. **Think in frames:** What updates every frame? Every 4? Every 8?
4. **Prioritize objects:** What needs MOBs now? What can wait?
5. **Keep it simple:** Constraints reward pragmatic code over abstractions

## Building and Running

```bash
# Compile
intybasic game.bas game.asm

# Assemble
as1600 -o game.bin -l game.lst game.asm

# Run
jzintv game.bin                    # Normal
jzintv --jlp game.bin              # With JLP support
jzintv -v1 game.bin                # With Intellivoice

# Debug
as1600 -j game.smap -s game.sym -o game.bin -l game.lst game.asm
intysmap game.smap
jzintv -d game.bin --src-map=game.smap --sym-file=game.sym
```

## Special Features

**Fixed-point numbers (8.8 format):**
```basic
#pos = 10.5           ' $0A80 in memory
#pos = #pos +. 0.25   ' Fixed add (+. operator)
y = #pos AND 255      ' Extract integer part
```

**Scrolling:**
```basic
SCROLL offset_x, offset_y, direction  
REM direction: 1=left, 2=right, 3=up, 4=down
REM Sprite positions need correction (see manual)
```

**Memory access:**
```basic
value = PEEK($1FF)    ' Read memory
POKE $200, value      ' Write memory
#ptr = VARPTR array   ' Get pointer to variable/array
```

**Assembly integration:**
```basic
result = USR my_func(arg1, arg2)  ' Call asm with return
CALL my_func(arg1, arg2)          ' Call asm no return
ASM INCLUDE "mycode.asm"          ' Include asm file
```

## When Writing IntyBASIC

- IntyBASIC auto-detects NTSC/PAL and adjusts timing
- Use OPTION EXPLICIT to catch typos in variable names
- Use STACK_CHECK during development to catch stack overflow
- WAIT synchronizes with video frame (use for timing)
- FRAME returns current frame number (useful for timing)
- COL0-COL7 only valid after WAIT (collision registers)
- Set CONT.KEY to 12 before waiting for keypress (debounce)
