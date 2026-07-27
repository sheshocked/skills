---
name: reverse-engineering
description: 
category: security
tags: [reverse-engineering]
---

## When to Use
Use this skill when performing binary reverse engineering — static analysis with Ghidra/radare2, dynamic analysis with GDB/LLDB, decompilation, binary patching, and anti-debug detection.

## Core Concepts
- **Static analysis**: Examine binary without execution (disassembly, decompilation)
- **Dynamic analysis**: Observe execution in debugger (breakpoints, memory inspection)
- **Decompilation**: Convert assembly to pseudo-C for readability
- **Binary patching**: Modify instructions/bytes to alter behavior
- **Anti-debug**: Techniques to detect and bypass debugger presence
- **ABI conventions**: x86-64 System V (Linux) vs Microsoft x64 (Windows)

## Workflow
1. **Identify binary type**: `file binary`, `binwalk binary`, check ELF/PE headers
2. **Static analysis**: Load in Ghidra, identify main function, find interesting strings
3. **Symbol resolution**: Resolve imports, cross-references, function boundaries
4. **Dynamic analysis**: Set breakpoints, trace execution, examine memory
5. **Decompile**: Convert critical functions to pseudo-C
6. **Patch**: Modify behavior if required (bypass checks, fix bugs)
7. **Document**: Record findings, offsets, and patch locations

## Key Patterns

### Ghidra — Automated Analysis
```bash
# Launch Ghidra headless analysis
analyzeHeadless /tmp/ghidra_project ProjectName \
  -import /path/to/binary \
  -postScript DecompileAllFunctions.java \
  -scriptPath /path/to/scripts \
  -deleteProject

# Create custom Ghidra script (Python/Java)
# In Ghidra Script Manager → new script
# Save as AnalyzeBinary.java

# Export decompiled functions
# Script → Export → Function Decompilation → save as .c files
```

### radare2 — Command-Line Analysis
```bash
# Open binary and analyze
r2 -A binary
# or
r2 -d binary  # debug mode

# Essential radare2 commands
aaa              # analyze all symbols
afl              # list functions
pdf @main        # disassemble main function
axt @sym.check_password  # cross-references to function
iz               # list strings
ii               # list imports
axt @entry0      # references to entry point

# Search for patterns
/ ad00          # search for byte pattern
/ad "flag{"     # search for string

# Patch binary
wa nop          # write NOP at current address
wao ret         # replace current instruction with RET
wx 9090         # write hex bytes

# Write patched binary
r2 -w binary    # open in write mode, then patch
```

### GDB — Dynamic Analysis
```bash
# Start GDB with binary
gdb ./binary

# Essential GDB commands
b main                    # breakpoint at main
b *0x401234               # breakpoint at address
b *check_password+42      # breakpoint at offset in function
r arg1 arg2               # run with arguments
n                         # next instruction (step over)
s                         # step into function
c                         # continue execution
x/20x $rsp               # examine 20 hex words at stack pointer
x/s 0x401234             # examine string at address
info registers            # show all registers
p $rax                    # print register value
set $rax = 1              # modify register
vmmap                     # show memory map
telescope 20              # show 20 stack values (GDB Enhanced Features)

# GDB Enhanced Features (GEF/pwndbg)
checksec                  # check binary protections
heap chunks               # show heap state
heap bins                 # show free bins
got                       # show GOT entries
canary                    # show stack canary value
```

### Anti-Debug Bypass
```bash
# Common anti-debug checks to look for
# ptrace(PTRACE_TRACEME, 0) — detects debugger
# /proc/self/status — check TracerPid
# /proc/self/maps — check for debugger mappings
# time() / RDTSC — timing checks
# IsDebuggerPresent() — Windows API

# Bypass ptrace check (Linux)
# Find ptrace call in disassembly
r2 -q -c "aa; /ad ptrace" binary

# Patch: replace ptrace call with NOP
r2 -q -c "aa; s <addr>; wa nop; wa nop; wa nop; wa nop; wa nop; wa nop; wa nop; wa nop; wa nop; wa nop" binary

# Or use LD_PRELOAD to intercept
cat > bypass.c << 'EOF'
#include <sys/ptrace.h>
long ptrace(int request, int pid, void *addr, void *data) {
    return 0;
}
EOF
gcc -shared -o bypass.so bypass.c -fPIC
LD_PRELOAD=./bypass.so ./binary
```

### Binary Patching Example
```bash
# Scenario: bypass license check
# Original code:
#   cmp eax, 1
#   je 0x401234  (jump if valid)
#   call fail_function

# Find the comparison
r2 binary
aaa
/pdf @main  # find the cmp instruction

# Patch: change je to jne (invert condition)
# je = 0x74, jne = 0x75
# Or NOP out the je to skip the check
wao nop  # NOP the conditional jump
# Or: wa nop; wa nop; wa nop  # NOP the je instruction (2 bytes)

# Alternative: patch the comparison
# cmp eax, 1 → cmp eax, 0
wx 83f800  # CMP EAX, 0

# Save patched binary
r2 -w binary
```

## Pitfalls
- **Stripped binaries**: No symbols — use function boundary detection and string references
- **Anti-debug complexity**: Multiple layers of protection — patch one, find another
- **ASLR/PIE**: Addresses change per run — use `echo 0 > /proc/sys/kernel/randomize_va_space` for debugging
- **Architecture mismatch**: Confirm x86 vs ARM vs MIPS before analysis
- **Legal issues**: Reverse engineering may violate DMCA/EULA — ensure legal authorization
- **Obfuscation**: Control flow flattening, string encryption — may need deobfuscation tools

## Verification
- `checksec --file=binary` — document protections (NX, PIE, RELRO, canary)
- Decompiled functions match original behavior
- Breakpoints trigger at expected locations
- Patched binary runs correctly with modified behavior
- No crashes or segfaults after patching
- All findings documented with file offsets and assembly instructions