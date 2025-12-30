# Intellivision Development Skills for Claude

Custom skills for Claude AI to assist with Intellivision game development, both high-level (IntyBASIC) and low-level (assembly/hardware).

## What Are These?

These are **Claude Skills** - knowledge packages that extend Claude's capabilities with specialized domain expertise. Upload them to claude.ai to get expert-level assistance with Intellivision development.

## Skills Included

### 1. IntyBASIC (`intybasic/`)

**Use for:** Writing, debugging, and optimizing IntyBASIC code

**Covers:**
- Complete IntyBASIC syntax reference
- MOB/sprite management patterns
- Graphics (BITMAP, DEFINE, GRAM/GROM)
- Input handling (controllers, keypads)
- Sound and music (PSG, tracker)
- Scrolling techniques
- Collision detection
- Optimization patterns
- Classic techniques (dynamic MOB assignment, frame spreading)
- File organization best practices

**Triggers when you:**
- Ask about IntyBASIC syntax
- Request code examples
- Mention MOBs, GRAM, cards, or STIC
- Need help with game logic
- Want optimization advice

### 2. Intellivision Hardware (`intellivision-hardware/`)

**Use for:** Assembly programming, reverse engineering, hardware-level development

**Covers:**
- CP1610 CPU instruction set (complete with cycle counts)
- STIC graphics chip (MOBs, BACKTAB, collision, scrolling)
- PSG sound chip (AY-3-8914, all registers)
- GROM character tables (ASCII mappings)
- EXEC ROM entry points
- Memory maps
- Assembly programming patterns
- Performance optimization
- Hardware timing and constraints

**Triggers when you:**
- Work with CP1610 assembly
- Ask about STIC registers
- Need hardware specifications
- Reverse engineer games
- Debug low-level code
- Optimize for performance

## Installation

### Upload to claude.ai (Recommended)

1. **Package the skills** (if modified):
   ```bash
   # Package IntyBASIC skill
   cd intybasic
   zip -r ../intybasic.skill SKILL.md
   cd ..
   
   # Package hardware skill
   cd intellivision-hardware
   zip -r ../intellivision-hardware.skill SKILL.md
   cd ..
   ```

2. **Upload to Claude:**
   - Go to [claude.ai](https://claude.ai)
   - Click Settings → Skills
   - Click "Add Skill"
   - Upload `intybasic.skill` and `intellivision-hardware.skill`

3. **Use them:**
   - Skills auto-trigger based on your questions
   - Or explicitly: "Using the IntyBASIC skill, help me..."

### Use with Claude Code (Alternative)

Skills aren't supported in Claude Code yet, but you can use the reference files:

1. Copy reference files to your project:
   ```bash
   cp INTYBASIC_REFERENCE.md /path/to/your/project/
   cp INTELLIVISION_HARDWARE_REFERENCE.md /path/to/your/project/
   ```

2. Claude Code will see them and reference them when working in that directory

## File Structure

```
.
├── README.md                              (this file)
│
├── intybasic/
│   └── SKILL.md                          (IntyBASIC skill source)
├── intybasic.skill                       (packaged for upload)
│
├── intellivision-hardware/
│   └── SKILL.md                          (Hardware skill source)
├── intellivision-hardware.skill          (packaged for upload)
│
├── INTYBASIC_REFERENCE.md                (Quick ref for Claude Code)
└── INTELLIVISION_HARDWARE_REFERENCE.md   (Quick ref for Claude Code)
```

## Editing Skills

### Modify Existing Skills

1. **Edit the source:**
   ```bash
   # Edit the markdown
   vim intybasic/SKILL.md
   ```

2. **Re-package:**
   ```bash
   cd intybasic
   zip -r ../intybasic.skill SKILL.md
   cd ..
   ```

3. **Re-upload:**
   - Go to claude.ai → Settings → Skills
   - Remove old version
   - Upload new `.skill` file

### What to Edit

**Frontmatter (YAML at top):**
```yaml
---
name: intybasic
description: When Claude should use this skill and what it covers
---
```
The `description` field is critical - it determines when the skill triggers.

**Body (Markdown):**
- Add new techniques
- Update examples
- Fix errors
- Add your own patterns

**Note:** Keep SKILL.md well-formed - invalid YAML or broken markdown may prevent the skill from loading in claude.ai.

## Using Both Skills Together

Both skills can be active simultaneously:

**Example use cases:**
- **"Write IntyBASIC code for scrolling"** → IntyBASIC skill loads
- **"What's the CP1610 instruction for shifting?"** → Hardware skill loads
- **"Explain how IntyBASIC generates MOB code"** → Both skills load

They complement each other:
- IntyBASIC skill for high-level programming
- Hardware skill for understanding what's happening underneath
- Together for optimization and debugging

## Reference Files (for Claude Code)

The `.md` reference files are condensed versions for quick lookup:

**INTYBASIC_REFERENCE.md:**
- Quick syntax guide
- Common patterns
- Essential techniques
- No fluff, just what you need

**INTELLIVISION_HARDWARE_REFERENCE.md:**
- CPU instructions
- Hardware registers
- Memory maps
- Assembly patterns

Keep these in your project root for Claude Code to reference.

## Skill Design Principles

These skills follow best practices:

1. **Concise** - Only essential information, no bloat
2. **Actionable** - Code examples, not just theory
3. **Accurate** - Based on official manuals and hardware docs
4. **Practical** - Real-world patterns that actually work
5. **Progressive** - Start simple, add detail as needed

## Technical Notes

### What's in a .skill File?

`.skill` files are ZIP archives containing:
```
intybasic.skill (ZIP)
  └── intybasic/SKILL.md
```

To extract:
```bash
unzip intybasic.skill
```

To inspect without extracting:
```bash
unzip -l intybasic.skill
```

### Skill Triggering

Skills trigger based on:
- Keywords in user messages (MOB, GRAM, IntyBASIC, CP1610, etc.)
- Context of conversation
- Explicit invocation ("using the X skill...")

Multiple skills can be active in one conversation.

### Context Window

Skills are loaded into Claude's context window when triggered. Keep them focused to avoid wasting tokens.

Current sizes:
- IntyBASIC: ~15KB
- Hardware: ~30KB

Both are well within limits for simultaneous use.

## Contributing

These skills are living documents. Improvements welcome:

**Add:**
- New IntyBASIC patterns you discover
- Optimizations from classic games
- Assembly tricks
- Hardware quirks

**Fix:**
- Errors in syntax
- Outdated information
- Unclear examples

**Update:**
- IntyBASIC version changes
- New compiler features
- Better examples

## Credits

**Skills created for:**
- IntyBASIC development (Oscar Toledo G.'s compiler)
- Intellivision homebrew programming
- Reverse engineering classic games
- Teaching retro game development

**Based on:**
- IntyBASIC v1.5.1 manual
- Joe Zbiciak's hardware documentation (spatula-city.org)
- Intellivision Programming documentation
- Real-world game development experience

**Hardware:**
- Intellivision console (1979)
- CPU: General Instrument CP1610 @ ~894 kHz
- Graphics: STIC chip
- Sound: General Instrument AY-3-8914 PSG

## AI Assistance Statement

**AIA PAI Nc Hin R Claude Sonnet 4.5 v1.0**

**AIA:** Primarily AI, New content, Human-initiated, Reviewed, Claude Sonnet 4.5 v1.0

This work was primarily AI-generated. AI was used to make new content, such as text, images, analysis, and ideas. AI was prompted for its contributions, or AI assistance was enabled. AI-generated content was reviewed and approved. The following model(s) or application(s) were used: Claude Sonnet 4.5.

**Co-Authored-By:** Claude (Anthropic AI)  
**Vibe-Coder:** Andrew Potozniak <tyraziel@gmail.com> (1.z3r0)

## License

MIT License - Applies to Skills and md files within this claude directory only.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

These skills document publicly available information about:
- IntyBASIC language (open source compiler)
- Intellivision hardware (publicly documented)
- Programming techniques (common knowledge)

The SKILL.md files themselves are provided as-is for educational purposes.

## Resources

**IntyBASIC:**
- Compiler: http://nanochess.org/
- Manual: https://github.com/nanochess/IntyBASIC

**Intellivision Hardware:**
- Joe Zbiciak's docs: http://spatula-city.org/~im14u2c/intv/
- jzintv emulator: http://spatula-city.org/~im14u2c/intv/
- AtariAge forums: https://atariage.com/forums/forum/112-intellivision/

**Development Tools:**
- as1600 assembler (included with jzintv)
- IntyBASIC compiler
- jzintv emulator/debugger

---

**Version:** 1.0  
**Updated:** December 2025  
**Compatible with:** IntyBASIC v1.5.1, Claude.ai skills system

Happy Intellivision development! 🎮
