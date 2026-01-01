# Miden Assembly Skill

A skill for understanding and writing Miden VM assembly (MASM) code.

## What's Included

- **SKILL.md** - Main skill file with quick reference, common patterns, and best practices
- **references/instruction_reference.md** - Complete instruction set with stack effects and cycle counts
- **references/code_organization.md** - Procedures, modules, constants, types, execution contexts
- **references/core_library.md** - Standard library modules (u64, hashes, memory, etc.)

## Installation

### Claude Code

Copy/symlink/checkout this skill folder to your Claude Code skills directory:

```
~/.claude/skills/
```

The skill will be automatically available in your next Claude Code session.

### Codex CLI (OpenAI)

Copy/symlink/checkout this skill folder to your Codex skills directory:

```
~/.codex/skills
```

Run Codex with skills enabled:

```bash
codex --enable skills
```

For other agents check out https://agentskills.io/home

## Usage

The skill activates when you:
- Read, analyze, or explain MASM code
- Write new MASM procedures or programs
- Debug MASM execution or stack state
- Optimize MASM code for cycle count
- Work with Miden VM stack machine concepts


## Skill evals

On the following request:

Can you spot the bug in this MASM code?

```masm
#! Saves two words to procedure locals and returns their hash.
@locals(9)
proc save_and_hash
    # Store words at locals 
    loc_storew_be.1 dropw
    loc_storew_be.5 dropw

    # Load both back and hash them
    padw loc_loadw_be.5
    padw loc_loadw_be.1
    hmerge
end

begin
    push.1.2.3.4
    push.5.6.7.8
    exec.save_and_hash
end
```

### Vanilla Opus response was:

```
⏺ Looking at this MASM code, I can spot the bug.

  The issue is with @locals(9) - you've allocated 9 local slots (indices 0-8), but you're storing at indices 1-4 and 5-8, which means you're wasting local slot 0.
  ...
```

Which means that it did not see the bug.

### Opus with this skill activated responded with:
```
⏺ Looking at this code, I can spot the bug: the local indices for word operations are not aligned to multiples of 4.

  From the Miden Assembly reference:
  Word operations (index must be multiple of 4)
  padw loc_storew_be.4    # Store word at local[4..7]

  The code uses loc_storew_be.1 and loc_storew_be.5, but word operations require indices that are multiples of 4 (0, 4, 8, etc.).
  ...
```

Which is correct. Also, note that it mentions the Miden Assembly reference from the skill.


