# Rev — vibe_checker.pyc (Python Bytecode Reverse)

## Challenge Information
- **Category**: Reverse Engineering
- **Difficulty**: Hard
- **Points**: Unknown
- **Flag**: `ZeroDays{v1b3_ch3ck_p4ss3d}`

## Description
A Python 3.10 `.pyc` file that's too new for standard decompilers (`decompyle3`, `uncompyle6`).

## Challenge Analysis

### The Problem
Python 3.10 introduced significant bytecode changes that break compatibility with older decompilers:
- `uncompyle6`: Supports up to Python 3.8
- `decompyle3`: Supports up to Python 3.8
- `pycdc`: May have issues with 3.10+

This forces us to analyze the bytecode directly.

## Solution

### Step 1: Load the Code Object
Use `xdis` (supports Python 3.10 bytecode) to load and inspect the code object:

```python
import xdis
from xdis.load import load_module

code_obj = load_module('vibe_checker.pyc')
```

### Step 2: Extract Constants
Examine the code object's constants:

```python
print(code_obj.co_consts)
```

**Key constants found:**
- `_VIBE_RING = '🎧✨🫠😎💀🔥🧠🌀'` (8 emoji characters)
- `_GLITCH_TAPE = 'd3✨a6✨42✨15✨6a✨e8✨9d✨f2✨9a✨cd✨99✨b0✨b5✨3e✨7f✨71✨6a✨70✨ab✨f5✨6e✨ed✨c7✨82✨82✨ef✨05'`

### Step 3: Analyze the Bytecode
Disassemble the `_vibe_mix()` function:

```python
import dis
dis.dis(code_obj)
```

**Critical finding:** Opcode `0x3f` appears in the algorithm:
- `0x3f` = `BINARY_RSHIFT` (>> operator, **not** left shift!)
- This is crucial for the decryption algorithm

### Step 4: Reconstruct the Algorithm
From bytecode analysis, the `_vibe_mix()` function:

```python
def _vibe_mix(input_bytes):
    _VIBE_RING = '🎧✨🫠😎💀🔥🧠🌀'
    output = []
    
    for i, byte in enumerate(input_bytes):
        mood = ord(_VIBE_RING[i % 8])
        spice = ((mood >> 1) ^ (i * 13) ^ 90) & 255  # Note: >> not <<
        output.append(byte ^ spice)
    
    return bytes(output)
```

**Key insight:** The algorithm uses **right shift** (`>> 1`), not left shift. This is determined by the bytecode opcode.

### Step 5: Parse the Ciphertext
The `_GLITCH_TAPE` contains hex values separated by ✨ emoji:

```python
ciphertext_hex = 'd3✨a6✨42✨15✨6a✨e8✨9d✨f2✨9a✨cd✨99✨b0✨b5✨3e✨7f✨71✨6a✨70✨ab✨f5✨6e✨ed✨c7✨82✨82✨ef✨05'
expected = [int(x, 16) for x in ciphertext_hex.split('✨')]
```

### Step 6: Decrypt the Flag
```python
_VIBE_RING = '🎧✨🫠😎💀🔥🧠🌀'
expected = [int(x,16) for x in 'd3✨a6✨42✨15✨6a✨e8✨9d✨f2✨9a✨cd✨99✨b0✨b5✨3e✨7f✨71✨6a✨70✨ab✨f5✨6e✨ed✨c7✨82✨82✨ef✨05'.split('✨')]

result = []
for i, enc in enumerate(expected):
    mood = ord(_VIBE_RING[i % 8])
    spice = ((mood >> 1) ^ (i * 13) ^ 90) & 255
    result.append(enc ^ spice)

flag = bytes(result).decode()
print(flag)
# ZeroDays{v1b3_ch3ck_p4ss3d}
```

## Complete Solution Script

```python
# Constants extracted from bytecode analysis
_VIBE_RING = '🎧✨🫠😎💀🔥🧠🌀'
_GLITCH_TAPE = 'd3✨a6✨42✨15✨6a✨e8✨9d✨f2✨9a✨cd✨99✨b0✨b5✨3e✨7f✨71✨6a✨70✨ab✨f5✨6e✨ed✨c7✨82✨82✨ef✨05'

# Parse ciphertext
expected = [int(x, 16) for x in _GLITCH_TAPE.split('✨')]

# Decrypt
result = []
for i, enc in enumerate(expected):
    # Get emoji character from ring (cycles every 8)
    mood = ord(_VIBE_RING[i % 8])
    
    # Calculate XOR key (CRITICAL: >> not <<)
    spice = ((mood >> 1) ^ (i * 13) ^ 90) & 255
    
    # XOR to decrypt
    result.append(enc ^ spice)

# Convert to string
flag = bytes(result).decode()
print(f"Flag: {flag}")
# Flag: ZeroDays{v1b3_ch3ck_p4ss3d}
```

## Technical Details

### Python 3.10 Bytecode Changes
Python 3.10 introduced several bytecode modifications:
- New opcodes for pattern matching
- Optimized exception handling
- Modified instruction format
- Breaking changes for decompilers

### Bytecode Opcode 0x3f
- **Opcode**: `0x3f`
- **Name**: `BINARY_RSHIFT`
- **Operation**: `>>` (right shift)
- **Not**: `BINARY_LSHIFT` (left shift)

This distinction is critical - using left shift instead of right shift produces garbage output.

### The Encryption Algorithm
```python
# For each byte at index i:
mood = ord(emoji_char[i % 8])     # Get Unicode value of emoji
spice = ((mood >> 1) ^ (i * 13) ^ 90) & 255  # Calculate XOR key
ciphertext[i] = plaintext[i] ^ spice  # XOR encryption
```

**Properties:**
- Position-dependent (uses index `i`)
- Emoji-based key derivation
- Multiple XOR operations for key mixing

### Emoji Unicode Values
The emojis in `_VIBE_RING` have high Unicode values:
- 🎧 = U+1F3A7 (127911)
- ✨ = U+2728 (10024)
- 🫠 = U+1FAE0 (129504)
- etc.

After right shift by 1, these become the base for the XOR key.

## Alternative Approaches

### 1. Dynamic Analysis
Run the `.pyc` file with Python 3.10:
```python
import vibe_checker
# Examine variables and functions
print(vibe_checker._VIBE_RING)
print(vibe_checker._GLITCH_TAPE)
```

### 2. Partial Decompilation
Some newer tools may partially decompile Python 3.10:
- `pycdc` (with latest updates)
- `decompyle3` (development version)

### 3. Manual Reconstruction
Read the bytecode instruction by instruction and reconstruct the source code manually.

## Common Pitfalls

### ❌ Wrong: Using Left Shift
```python
spice = ((mood << 1) ^ (i * 13) ^ 90) & 255  # WRONG!
```
This produces incorrect output because the bytecode clearly shows `BINARY_RSHIFT`.

### ✅ Correct: Using Right Shift
```python
spice = ((mood >> 1) ^ (i * 13) ^ 90) & 255  # CORRECT
```

## Lessons Learned

1. **Bytecode analysis is essential**: When decompilers fail, read bytecode directly
2. **Opcodes matter**: Confusing >> and << produces wrong results
3. **Version compatibility**: Python bytecode changes between versions
4. **Unicode in crypto**: Emojis can be used as key material
5. **XOR is reversible**: XOR encryption is its own inverse

## Tools Used
- **xdis**: Python bytecode disassembler supporting 3.10+
- **dis module**: Built-in Python disassembler
- **Python 3.10+**: For running the bytecode
- **Hex editor**: For examining the .pyc file structure

## References
- [Python Bytecode](https://docs.python.org/3/library/dis.html)
- [xdis Documentation](https://github.com/rocky/python-xdis)
- [Python 3.10 What's New](https://docs.python.org/3/whatsnew/3.10.html)
- [Unicode Emoji](https://unicode.org/emoji/charts/full-emoji-list.html)

---

**Flag**: `ZeroDays{v1b3_ch3ck_p4ss3d}`

**Solved**: ✅

**Key Takeaway**: When Python decompilers fail due to version incompatibility, bytecode analysis is the solution. Pay careful attention to opcodes - confusing similar operations (like >> vs <<) will produce incorrect results.
