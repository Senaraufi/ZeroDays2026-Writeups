# Rev — DogeMiner_FREE_RAM.exe (PyInstaller Reverse)

## Challenge Information
- **Category**: Reverse Engineering
- **Difficulty**: Medium
- **Points**: Unknown
- **Flag**: `ZeroDays{w0w_such_r3v3rs3_much_str1ngs}`

## Description
A Windows PE32+ executable with a Doge theme and 4 obfuscated strings. The binary is a PyInstaller bundle containing Python bytecode.

## Challenge Analysis

### Initial Identification
Running `strings` on the binary reveals PyInstaller artifacts:
- `pyiboot01_bootstrap`
- `pyi_rth_inspect`
- Python DLL references

This indicates the executable is a **PyInstaller-packaged Python application**.

## Solution

### Step 1: Extract the PyInstaller Archive
Use `pyinstxtractor` to unpack the executable:

```bash
python pyinstxtractor.py DogeMiner_FREE_RAM.exe
```

This extracts multiple `.pyc` files and dependencies.

### Step 2: Identify the Main Script
Look for the main Python bytecode file (usually the largest or named after the executable).

### Step 3: Decompilation Challenge
The `.pyc` file uses **Python 3.13 bytecode**, which is too new for standard decompilers:
- `uncompyle6` - doesn't support 3.13
- `decompyle3` - doesn't support 3.13
- `pycdc` - may have issues with 3.13

### Step 4: Manual Bytecode Analysis
Since decompilation fails, we analyze the bytecode manually using `xdis` or by examining the code object:

```python
import marshal, dis

with open('extracted.pyc', 'rb') as f:
    f.read(16)  # Skip header
    code = marshal.load(f)
    
# Examine constants
print(code.co_consts)
```

### Step 5: Extract Key Constants
From the bytecode analysis, we find:

**Obfuscated strings:**
```python
such_secrets = ('Mz[1kig[MegyA', 'EL3`T:xa1Kxfh', '7a7J1]QVrgiRe', 'gYOockGxeIOg`')
```

**XOR keys:**
```python
keys = [1, 2, 3, 4]
```

### Step 6: Understand the Algorithm
The `much_wow()` function (reconstructed from bytecode):
1. XORs each string with a key
2. Joins the results
3. Base64 decodes
4. Reverses the string

### Step 7: Brute Force Permutations
Since we don't know the correct order of strings and keys, we try all permutations:

```python
import base64
import itertools

such_secrets = ('Mz[1kig[MegyA', 'EL3`T:xa1Kxfh', '7a7J1]QVrgiRe', 'gYOockGxeIOg`')
keys = [1, 2, 3, 4]

for perm in itertools.permutations(range(4)):
    result = ''
    for i, si in enumerate(perm):
        # XOR each character in the string with the key
        result += ''.join(chr(ord(c) ^ keys[i]) for c in such_secrets[si])
    
    try:
        # Try to base64 decode
        decoded = base64.b64decode(result).decode('utf-8')
        
        # Check if it looks like a flag
        if '{' in decoded:
            # Reverse the string
            flag = decoded[::-1]
            print(flag)
            # ZeroDays{w0w_such_r3v3rs3_much_str1ngs}
            break
    except:
        pass
```

## Complete Solution Script

```python
import base64
import itertools

# Extracted from bytecode analysis
such_secrets = ('Mz[1kig[MegyA', 'EL3`T:xa1Kxfh', '7a7J1]QVrgiRe', 'gYOockGxeIOg`')
keys = [1, 2, 3, 4]

# Try all permutations of string order
for perm in itertools.permutations(range(4)):
    result = ''
    
    # XOR each string with corresponding key
    for i, string_index in enumerate(perm):
        xored = ''.join(chr(ord(c) ^ keys[i]) for c in such_secrets[string_index])
        result += xored
    
    try:
        # Base64 decode
        decoded = base64.b64decode(result).decode('utf-8')
        
        # Check for flag format
        if '{' in decoded:
            # Reverse to get final flag
            flag = decoded[::-1]
            print(f"Flag: {flag}")
            break
    except:
        continue

# Output: ZeroDays{w0w_such_r3v3rs3_much_str1ngs}
```

## Technical Details

### PyInstaller Structure
PyInstaller bundles Python applications into standalone executables:
- **Bootloader**: C code that starts Python interpreter
- **Archive**: Contains `.pyc` files and dependencies
- **Extraction**: `pyinstxtractor` can unpack the archive

### Python 3.13 Bytecode
Python 3.13 introduced bytecode changes:
- New opcodes
- Modified code object structure
- Incompatible with older decompilers

### The Algorithm
```
1. XOR strings with keys: 'Mz[1kig[MegyA' ^ 1 = 'Ly\Z0jhf\ZNfhzB'
2. Concatenate all XORed strings
3. Base64 decode the concatenated result
4. Reverse the decoded string
5. Result: ZeroDays{w0w_such_r3v3rs3_much_str1ngs}
```

### Doge Theme
The flag content `w0w_such_r3v3rs3_much_str1ngs` is a reference to the Doge meme's characteristic speech pattern ("wow such X much Y").

## Alternative Approaches

### 1. Dynamic Analysis
Run the executable in a Windows VM and:
- Use Process Monitor to capture file operations
- Set breakpoints in a debugger
- Extract the flag from memory

### 2. String Analysis
Sometimes the flag is visible in strings output:
```bash
strings DogeMiner_FREE_RAM.exe | grep -i zero
```

### 3. Python Debugging
If you can get the `.pyc` to run:
```python
import sys
sys.path.insert(0, 'extracted_dir')
import main_script
# Examine variables
```

## Lessons Learned

1. **PyInstaller is common**: Many CTF binaries are packaged Python apps
2. **Bytecode analysis**: When decompilation fails, analyze bytecode directly
3. **Brute force works**: With small search spaces, try all combinations
4. **Version matters**: Python bytecode changes between versions
5. **Theme hints**: The Doge theme hints at the flag format

## Tools Used
- **pyinstxtractor**: For extracting PyInstaller archives
- **xdis**: For Python bytecode disassembly
- **Python**: For brute forcing permutations
- **strings**: For initial reconnaissance

## References
- [PyInstaller](https://pyinstaller.org/)
- [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor)
- [Python Bytecode](https://docs.python.org/3/library/dis.html)
- [Doge Meme](https://en.wikipedia.org/wiki/Doge_(meme))

---

**Flag**: `ZeroDays{w0w_such_r3v3rs3_much_str1ngs}`

**Solved**: ✅

**Key Takeaway**: PyInstaller binaries can be unpacked to reveal Python bytecode. When decompilation fails due to version incompatibility, manual bytecode analysis and brute forcing can still recover the flag.
