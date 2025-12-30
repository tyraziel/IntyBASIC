---
name: intellivision-hardware
description: Comprehensive Intellivision hardware reference for assembly programming and reverse engineering. Use when working with CP1610 assembly, STIC graphics registers, PSG sound chip, GROM character tables, EXEC ROM routines, or analyzing/debugging low-level Intellivision code. Covers instruction set, hardware registers, memory maps, and classic programming patterns.
---

# Intellivision Hardware Reference

Complete hardware documentation for CP1610 CPU, STIC graphics, PSG sound, GROM, and EXEC ROM.

## CP1610 CPU

**Clock Speed:** ~894.886 kHz (NTSC), ~1 MHz (PAL)  
**Data Width:** 16-bit (decle - "sixteen-bit word")  
**Address Space:** 64K words (16-bit addressable)

### Registers

```
R0-R5: General purpose
R6 (SP): Stack Pointer
R7 (PC): Program Counter

Flag bits (in status):
  S: Sign
  Z: Zero  
  C: Carry
  O: Overflow
  I: Interrupt enable
  D: Double precision mode
```

### Key Instructions

**Data Movement:**
```
MVO  Rn, addr     ; Move Out - store register to memory
MVI  addr, Rn     ; Move In - load memory to register
MVII #imm, Rn     ; Move Immediate - load constant
MOVR Rm, Rn       ; Move Register to register
```

**Arithmetic:**
```
ADD  Rm, Rn       ; Rn = Rn + Rm
SUB  Rm, Rn       ; Rn = Rn - Rm
ADDI #imm, Rn     ; Rn = Rn + immediate
SUBI #imm, Rn     ; Rn = Rn - immediate
INCR Rn           ; Rn = Rn + 1
DECR Rn           ; Rn = Rn - 1
NEGR Rn           ; Rn = -Rn
COMR Rn           ; Rn = ~Rn (ones complement)
```

**Logic:**
```
ANDI #imm, Rn     ; Rn = Rn AND immediate
XORI #imm, Rn     ; Rn = Rn XOR immediate
AND  Rm, Rn       ; Rn = Rn AND Rm
XOR  Rm, Rn       ; Rn = Rn XOR Rm
```

**Shifts:**
```
SLL  Rn, 1        ; Shift left logical (×2)
SLL  Rn, 2        ; Shift left logical (×4)
SLR  Rn, 1        ; Shift right logical (÷2)
SLR  Rn, 2        ; Shift right logical (÷4)
SAR  Rn, 1        ; Shift arithmetic right (÷2, preserve sign)
SAR  Rn, 2        ; Shift arithmetic right (÷4, preserve sign)
SWAP Rn           ; Swap bytes
RLC  Rn           ; Rotate left through carry
RRC  Rn           ; Rotate right through carry
```

**Branches:**
```
B    addr         ; Branch unconditional
BC   addr         ; Branch if carry set
BNC  addr         ; Branch if carry clear
BOV  addr         ; Branch if overflow
BNOV addr         ; Branch if no overflow
BPL  addr         ; Branch if plus (S=0)
BMI  addr         ; Branch if minus (S=1)
BEQ  addr         ; Branch if zero (Z=1)
BNEQ addr         ; Branch if not zero (Z=0)
BLT  addr         ; Branch if less than (signed)
BGE  addr         ; Branch if greater/equal (signed)
BLE  addr         ; Branch if less/equal (signed)
BGT  addr         ; Branch if greater than (signed)
BUSC addr         ; Branch if unsigned less than (C=0)
BESC addr         ; Branch if unsigned greater/equal (C=1)
```

**Jumps/Calls:**
```
J    addr         ; Jump direct
JR   Rn           ; Jump register
JSR  Rn, addr     ; Jump subroutine (save PC in Rn)
JSRE Rn, addr     ; JSR with interrupt enable
JSRD Rn, addr     ; JSR with interrupt disable
```

**Stack Operations:**
```
PSHR Rn           ; Push register to stack
PULR Rn           ; Pull register from stack
```

**Special:**
```
NOP               ; No operation (MOVR R0,R0)
HLT               ; Halt
SDBD              ; Set Double Byte Data (next instruction uses 16-bit immediate)
SIN               ; Software interrupt
EIS               ; Enable interrupts
DIS               ; Disable interrupts
TCI               ; Terminate current interrupt
CLRC              ; Clear carry
SETC              ; Set carry
```

### Addressing Modes

```
Immediate:     MVII #$1234, R0
Direct:        MVI  $1234, R0
Indirect:      MVI@ R4, R0        ; Load from address in R4
Auto-inc:      MVI@ R4, R0        ; Then R4++
Auto-dec:      MVI@ R4, R0        ; Then R4--
Indexed:       MVI  $10(R4), R0   ; Load from R4+$10
```

### Cycle Counts (Critical for Performance)

```
MVI:  8 cycles (direct), 7 cycles (indirect)
MVO:  9 cycles (direct), 9 cycles (indirect)
MVII: 8 cycles
ADD:  6 cycles
MULT: Variable (custom routines, 42-702 cycles)
DIV:  Variable (custom routines, 200+ cycles)
Branch taken: 7-9 cycles
Branch not taken: 7 cycles
```

### Memory Map

```
$0000-$003F   STIC registers (write during vertical blank)
$0040-$00FF   System RAM (scratch)
$0100-$01EF   System RAM (8-bit variables)
$01F0-$01FF   System RAM (controller input)
$0200-$035F   BACKTAB (screen memory, 20×12)
$0360-$03FF   System RAM
$0400-$04FF   Graphics RAM (GRAM) - optional
$0500-$xxxx   Cartridge ROM (varies by map)
$D000-$DFFF   GROM (Graphics ROM)
$F000-$FFFF   EXEC ROM (operating system)
```

## STIC (Standard Television Interface Chip)

Graphics controller managing display, sprites (MOBs), and collision detection.

### MOB (Moving OBject) Registers

**8 MOBs total (MOB 0-7), each has 3 registers:**

```
$0000-$0007: MOB X positions
$0008-$000F: MOB Y positions  
$0010-$0017: MOB attributes (card + color)
```

**X Register ($0000-$0007):**
```
Bits 0-7:   X position (0-255, display 0-168)
Bit 8:      Interaction (1=collision detection enabled)
Bit 9:      Visibility (1=visible)
Bit 10:     Double width (1=16 pixels wide)
Bits 11-15: Unused
```

**Y Register ($0008-$000F):**
```
Bits 0-6:   Y position (0-127, display 0-95)
Bit 7:      Double height (1=16 lines, must use even GRAM card)
Bits 8-9:   Vertical magnification
            00 = ×0.5 (4 lines)
            01 = ×1   (8 lines)  
            10 = ×2   (16 lines)
            11 = ×4   (32 lines)
Bit 10:     X flip (mirror horizontal)
Bit 11:     Y flip (mirror vertical)
Bits 12-15: Unused
```

**Attribute Register ($0010-$0017):**
```
Bits 0-2:   Color bits 0-2
Bits 3-11:  Card number (0-511)
            0-255:   GROM cards
            256-319: GRAM cards (if defined)
            320-511: GROM repeated
Bit 12:     Color bit 3 (high bit)
Bit 13:     Priority (0=foreground, 1=background)
Bits 14-15: Unused

Colors (4-bit):
  0=$0: Black       8=$1: Blue
  1=$2: Blue        9=$3: Red  
  2=$4: Red         A=$5: Tan
  3=$6: Tan         B=$7: Dark green
  4=$8: Dark green  C=$9: Green
  5=$A: Green       D=$B: Yellow
  6=$C: Yellow      E=$D: White
  7=$E: White       F=$F: White
```

### BACKTAB (Screen Memory)

**Location:** $0200-$035F (240 words = 20 cols × 12 rows)

**Card Format (Color Stack Mode):**
```
Bits 0-2:   Foreground color (0-7)
Bits 3-11:  Card number (0-511, same as MOB)
Bit 12:     Foreground color bit 3
Bit 13:     Advance color stack pointer
Bit 14:     GROM/GRAM select
Bit 15:     Unused
```

**Card Format (Foreground/Background Mode):**
```
Bits 0-3:   Foreground color (0-15)
Bits 3-7:   Card number (0-63, limited set)
Bits 8-11:  Background color (0-15)
Bit 12:     GROM/GRAM select
Bits 13-15: Unused
```

### Display Control Registers

```
$0020: Display enable/mode
  Bit 0: Display enable
  Bit 1: Mode (0=color stack, 1=foreground/background)

$0021: Color stack 0
$0022: Color stack 1
$0023: Color stack 2  
$0024: Color stack 3

$0028: Horizontal delay (0-7 pixels)
$0029: Vertical delay (0-7 pixels)

$002C: Border color (0-15)
$0030: Border extension
  Bit 0: Left column masked
  Bit 1: Top row masked
```

### Display Resolution and Scrolling

**BACKTAB Size:** 20 columns × 12 rows = 160×96 pixels (each tile 8×8)

**Visible Display:**
- Horizontal: ~159 pixels (essentially full 20 tiles)
- Vertical: ~88-96 pixels (11-12 tiles, varies by TV overscan)

**Smooth Scrolling Technique:**

The STIC scroll registers ($0028, $0029) allow pixel-perfect scrolling by offsetting the display 0-7 pixels in any direction. The BACKTAB content remains unchanged until you cross an 8-pixel boundary.

**Horizontal scrolling algorithm:**
```
1. Set horizontal delay 0-7 ($0028)
2. When delay reaches 8:
   - Reset delay to 0
   - Shift BACKTAB content left/right by 1 column
   - Update edge column with new tiles
```

**Example - scrolling right:**
```
Frame 1: delay=0, BACKTAB shows columns [A,B,C...T]
Frame 2: delay=1, same BACKTAB, shifted 1 pixel right
Frame 3: delay=2, same BACKTAB, shifted 2 pixels
...
Frame 8: delay=7, same BACKTAB, shifted 7 pixels
Frame 9: delay=0, BACKTAB now [B,C,D...U], column A scrolled off, U scrolled in
```

**CRITICAL: MOB Position Correction**

Scroll registers affect BACKTAB only, NOT MOBs. When scrolling, you must manually adjust all MOB positions to compensate:

```asm
; Scrolling right by scroll_offset pixels
; MOBs must move LEFT by same amount to stay in place

; For each MOB:
MOB_X_adjusted = MOB_X_world - scroll_offset
```

For typical side-scroller:
- Maintain world coordinates for all objects
- Calculate screen position = world_position - scroll_offset
- Only display MOBs within visible range
- Update BACKTAB window when crossing tile boundaries

### Collision Detection Registers

**Location:** $0018-$001F (read after vertical blank)

```
$0018: COL0 - MOB 0 collision status
$0019: COL1 - MOB 1 collision status
...
$001F: COL7 - MOB 7 collision status

Collision bits:
  Bits 0-7: Collision with MOB 0-7
  Bit 8:    Collision with background (any set pixel)
  Bit 9:    Collision with screen border
  Bits 10-15: Unused
```

### GRAM (Graphics RAM)

**Location:** $3000-$33FF (1024 words, optional)  
**Cards:** 256-319 (64 cards × 8 bytes each)

Each card is 8×8 pixels, defined by 8 bytes (one per row).

**Example - Define Card 256 (letter "A"):**
```
$3000: %00011000  ; Row 0:    XX
$3001: %00100100  ; Row 1:   X  X
$3002: %01000010  ; Row 2:  X    X
$3003: %01111110  ; Row 3:  XXXXXX
$3004: %01000010  ; Row 4:  X    X
$3005: %01000010  ; Row 5:  X    X
$3006: %01000010  ; Row 6:  X    X
$3007: %00000000  ; Row 7:
```

### Critical Timing

**NTSC:** 60 Hz, 16.67ms per frame  
**PAL:** 50 Hz, 20ms per frame

**Vertical Blank Period:**
- Lasts ~20-25 scanlines (~1.3ms NTSC)
- ONLY time you can safely write to STIC registers
- Writing outside VBLANK → screen artifacts

**GRAM Loading:**
- ~18 cards max per frame (NTSC)
- Reduced to 16 with music player
- Reduced to 13 with full ECS music

## PSG (Programmable Sound Generator - AY-3-8914)

3 tone channels + 1 noise channel with envelope generator.

### Registers (accessed via $01F0)

**Tone Generators:**
```
$00-$01: Channel A frequency (12-bit)
$02-$03: Channel B frequency (12-bit)  
$04-$05: Channel C frequency (12-bit)

Frequency calculation:
  NTSC: freq_value = 3579545 / (32 × desired_hz)
  PAL:  freq_value = 4000000 / (32 × desired_hz)

Examples (NTSC):
  A4 (440 Hz):  freq = 3579545 / (32 × 440) = 254
  C5 (523 Hz):  freq = 3579545 / (32 × 523) = 214
```

**Noise Generator:**
```
$06: Noise period (5-bit, 0-31)
  Lower = higher pitch noise
```

**Mixer Control:**
```
$07: Enable/disable (0=on, 1=off for each bit)
  Bit 0: Tone A enable
  Bit 1: Tone B enable
  Bit 2: Tone C enable
  Bit 3: Noise A enable
  Bit 4: Noise B enable
  Bit 5: Noise C enable
  Bits 6-7: I/O port mode

Typical value: $38 = %00111000
  = All tones ON, all noise OFF
```

**Volume Control:**
```
$08: Channel A volume (0-15, or 16 for envelope)
$09: Channel B volume (0-15, or 16 for envelope)
$0A: Channel C volume (0-15, or 16 for envelope)

  0 = silence
  15 = maximum
  16 = use envelope generator
```

**Envelope Generator:**
```
$0B-$0C: Envelope period (16-bit)
$0D: Envelope shape (0-15)

Common shapes:
  0: \______  Decay to zero
  4: /|/|/|/  Triangle wave
  8: \\\\\\  Repeated decay
  10: \/\/\/  Triangle (alternate)
  11: /‾‾‾‾‾  Attack and hold
  14: /\/\/\  Sawtooth
```

### ECS (Entertainment Computer System) - Second PSG

Same registers, accessed differently. Allows 8 channels total (4 base + 4 ECS).

## GROM (Graphics ROM)

**Location:** $3000-$3FFF (GROM appears here when GRAM not enabled)  
**Cards:** 0-255 (built-in character set)

### GROM Character Table

**Control Characters (0-31):**
```
0:  Space/blank
1-26: (Various symbols in actual GROM)
27-31: (Various symbols)
```

**Numbers and Symbols (32-63):**
```
32:  (space)          48: 0
33:  !                49: 1
34:  "                50: 2
35:  #                51: 3
36:  $                52: 4
37:  %                53: 5
38:  &                54: 6
39:  '                55: 7
40:  (                56: 8
41:  )                57: 9
42:  *                58: :
43:  +                59: ;
44:  ,                60: <
45:  -                61: =
46:  .                62: >
47:  /                63: ?
```

**Uppercase Letters (64-95):**
```
64:  @                80: P
65:  A                81: Q
66:  B                82: R
67:  C                83: S
68:  D                84: T
69:  E                85: U
70:  F                86: V
71:  G                87: W
72:  H                88: X
73:  I                89: Y
74:  J                90: Z
75:  K                91: [
76:  L                92: \
77:  M                93: ]
78:  N                94: ^
79:  O                95: _
```

**Special/Lowercase (96-127):**
```
96:   `               112: (special)
97:   a               113-127: (various)
98-111: (lowercase continues)
```

**Extended Characters (128-255):**
- Card suits, game symbols, box drawing
- Varies by GROM version

### ASCII to GROM Conversion

IntyBASIC handles this automatically in PRINT statements, but for assembly:

```
ASCII 32-95  → GROM cards 32-95  (direct mapping)
ASCII 97-122 → GROM cards 97-122 (lowercase, if present)
```

## EXEC ROM Routines

**Location:** $1000-$1FFF (executive ROM)

Common entry points (documentation varies by EXEC version):

### Utility Routines

```
$xxxx: CLRSCR - Clear screen
  Input: None
  Output: Screen cleared
  Clobbers: R0-R4

$xxxx: PRINT - Print character
  Input: R0=card number, R4=screen position
  Output: Character on screen
  Clobbers: R0, R1

$xxxx: FILLMEM - Fill memory
  Input: R0=value, R1=count, R4=address
  Output: Memory filled
  Clobbers: R0, R1, R4

$xxxx: MULT16 - Multiply 16-bit
  Input: R0=multiplicand, R1=multiplier
  Output: R0=result (low), R1=result (high)
  Clobbers: R0, R1, R2

$xxxx: DIV16 - Divide 16-bit  
  Input: R0=dividend, R1=divisor
  Output: R0=quotient, R1=remainder
  Clobbers: R0, R1, R2
```

### Controller Input

```
$1xx: SCANHAND - Scan controllers
  Input: None
  Output: 
    $01FE: Right controller state
    $01FF: Left controller state
  Format (complemented):
    Bits 0-3: Disc direction (one-hot)
    Bits 4-7: Keypad/buttons

Controller bits (after complementing):
  Bit 0: East (right)
  Bit 1: South (down)
  Bit 2: West (left)
  Bit 3: North (up)
  Bit 4-6: Keypad row
  Bit 7: Action button
```

## Assembly Programming Patterns

### Efficient Multiplication

```asm
; Multiply by power of 2 - use shifts
SLL  R0, 1    ; ×2
SLL  R0, 2    ; ×4

; Multiply by small constant - use adds
ADD  R0, R0   ; ×2
ADD  R1, R0   ; ×3 (if R1=original)

; Variable × variable - use lookup table or algorithm
```

### Fast Division

```asm
; Divide by power of 2 - use shifts  
SLR  R0, 1    ; ÷2 (unsigned)
SAR  R0, 1    ; ÷2 (signed)

; Divide by constant - use multiply by reciprocal if possible
```

### MOB Management Pattern

```asm
; Set up MOB 0
MVII #$300, R0     ; X position + interaction + visibility
MVO  R0, $0000     ; Write X
MVII #$200, R0     ; Y position + 1x scale
MVO  R0, $0008     ; Write Y
MVII #CARD+COLOR, R0
MVO  R0, $0010     ; Write attribute

; Disable MOB
CLRR R0
MVO  R0, $0000
MVO  R0, $0008
MVO  R0, $0010
```

### VBLANK Synchronization

```asm
; Wait for vertical blank
WAIT_VBLANK:
  MVI  $0020, R0    ; Read display enable
  ANDI #1, R0       ; Check if display active
  BEQ  WAIT_VBLANK  ; Loop until VBLANK
  
; Now safe to write STIC registers
```

### Collision Detection

```asm
; Check MOB 0 collision
MVI  $0018, R0    ; Read COL0
ANDI #$FF, R0     ; Mask to MOB collision bits
BEQ  NO_HIT       ; Zero = no collision

; Check specific MOB
ANDI #$02, R0     ; Bit 1 = hit MOB 1
BNEQ HIT_MOB1

; Check background hit
MVI  $0018, R0
ANDI #$100, R0    ; Bit 8 = background
BNEQ HIT_BG
```

## Performance Optimization

### Critical Rules

1. **Use shifts not multiply/divide** for powers of 2
2. **Minimize memory access** - keep values in registers
3. **Unroll loops** for small fixed counts
4. **Use direct addressing** when possible (faster than indirect)
5. **Write STIC only during VBLANK** to avoid artifacts
6. **Batch GRAM updates** - max ~18 cards/frame

### Cycle Budget (NTSC)

```
Frame time: 14,934 cycles (16.67ms @ 894kHz)
VBLANK: ~1,000-1,500 cycles available

Typical frame breakdown:
  Controller scan: ~100 cycles
  Game logic: ~5,000 cycles
  STIC updates: ~500 cycles
  GRAM updates: ~200 cycles/card
  Sound updates: ~100 cycles
  Remaining: Reserve for peaks
```

## Common Pitfalls

1. **Writing STIC outside VBLANK** → screen glitches
2. **Too many GRAM updates** → can't load all, graphics flicker
3. **Forgetting to set interaction bit** → collisions don't work
4. **Wrong color bit mapping** → bit 12 is separate from 0-2
5. **Stack overflow** → R6 grows down, watch limits
6. **16-bit immediate without SDBD** → only gets low byte
7. **Collision check before WAIT** → stale data

## References

This skill provides hardware reference only. For actual ROM code or complete register documentation, see:
- spatula-city.org/~im14u2c/intv/ (Joe Zbiciak's comprehensive docs)
- Programming the Intellivision (Intellivision Productions book)
- as1600 assembler documentation (jzintv package)

---

**Hardware:** Intellivision console (1979)  
**CPU:** General Instrument CP1610 @ ~894 kHz  
**Graphics:** STIC (Standard Television Interface Chip)  
**Sound:** General Instrument AY-3-8914 PSG  
**This reference covers publicly documented hardware behavior and programming techniques.**
