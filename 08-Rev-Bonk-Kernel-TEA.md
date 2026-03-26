# Rev — bonk.sys (Kernel Driver TEA Decryption)

## Challenge Information
- **Category**: Reverse Engineering
- **Difficulty**: Hard
- **Points**: Unknown
- **Flag**: `ZeroDays{k3rn3lm0d3b0nk}`

## Description
A Windows kernel driver (`.sys` file) that implements a custom IOCTL requiring a magic value `0xDEADB04B` ("DEAD BONK").

## Challenge Analysis

### File Type
- **Format**: Windows Kernel Driver (.sys)
- **Architecture**: x64
- **Type**: PE32+ executable

### Initial Analysis
```bash
file bonk.sys
# bonk.sys: PE32+ executable (native) x86-64, for MS Windows
```

## Solution

### Step 1: Disassembly
Load the driver in a disassembler (IDA Pro, Ghidra, or Binary Ninja) and analyze the IOCTL handler.

### Step 2: Identify the IOCTL Code
The driver registers an IOCTL handler for code `0x222004`:
- Requires magic value `0xDEADB04B` in the input buffer
- Performs decryption on embedded ciphertext
- Returns the decrypted flag

### Step 3: Identify the Cipher
Analyzing the decryption routine reveals the **TEA (Tiny Encryption Algorithm)**:

**Signature patterns:**
- `shl eax, 4` and `shr eax, 5` operations
- 32-round loop
- Delta constant `0x9E3779B9`
- 128-bit key (4 × 32-bit words)

### Step 4: Extract the Key
The TEA key is stored in the `.data` section:
```c
uint32_t key[4] = {0x11223344, 0x55667788, 0x99aabbcc, 0xddeeff00};
```

### Step 5: Extract the Ciphertext
The ciphertext is loaded into the output buffer using `movups` and `movsd` instructions:
```
Ciphertext (24 bytes):
16 45 1e 64 b5 a9 42 61 00 89 89 cc 9a 0d aa 85 76 58 62 f0 22 6a 95 0d
```

### Step 6: Implement TEA Decryption
```python
import struct

def tea_decrypt(v0, v1, key):
    """
    TEA decryption algorithm
    v0, v1: 32-bit ciphertext blocks
    key: tuple of 4 32-bit key values
    """
    delta = 0x9E3779B9
    mask = 0xFFFFFFFF
    
    # Initial sum for decryption (delta * rounds)
    s = (delta * 32) & mask
    
    k0, k1, k2, k3 = key
    
    # 32 rounds of decryption
    for _ in range(32):
        v1 = (v1 - (((v0 << 4) + k2) ^ (v0 + s) ^ ((v0 >> 5) + k3))) & mask
        v0 = (v0 - (((v1 << 4) + k0) ^ (v1 + s) ^ ((v1 >> 5) + k1))) & mask
        s = (s - delta) & mask
    
    return v0, v1

# Key from .data section
key = (0x11223344, 0x55667788, 0x99aabbcc, 0xddeeff00)

# Ciphertext (24 bytes = 6 32-bit words = 3 TEA blocks)
ciphertext = bytes.fromhex('16451e64b5a94261008989cc9a0daa85765862f0226a950d')
blocks = struct.unpack('<6I', ciphertext)

# Decrypt each 64-bit block
result = b''
for i in range(0, 6, 2):
    v0, v1 = tea_decrypt(blocks[i], blocks[i+1], key)
    result += struct.pack('<II', v0, v1)

print(result.decode())
# ZeroDays{k3rn3lm0d3b0nk}
```

## Complete Solution Script

```python
import struct

def tea_decrypt(v0, v1, key):
    """TEA (Tiny Encryption Algorithm) decryption"""
    delta = 0x9E3779B9
    mask = 0xFFFFFFFF
    s = (delta * 32) & mask
    k0, k1, k2, k3 = key
    
    for _ in range(32):
        v1 = (v1 - (((v0 << 4) + k2) ^ (v0 + s) ^ ((v0 >> 5) + k3))) & mask
        v0 = (v0 - (((v1 << 4) + k0) ^ (v1 + s) ^ ((v1 >> 5) + k1))) & mask
        s = (s - delta) & mask
    
    return v0, v1

# Extracted from bonk.sys
key = (0x11223344, 0x55667788, 0x99aabbcc, 0xddeeff00)
ciphertext = bytes.fromhex('16451e64b5a94261008989cc9a0daa85765862f0226a950d')

# Parse as little-endian 32-bit integers
blocks = struct.unpack('<6I', ciphertext)

# Decrypt 3 blocks (24 bytes total)
plaintext = b''
for i in range(0, 6, 2):
    v0, v1 = tea_decrypt(blocks[i], blocks[i+1], key)
    plaintext += struct.pack('<II', v0, v1)

print(f"Flag: {plaintext.decode()}")
# Flag: ZeroDays{k3rn3lm0d3b0nk}
```

## Technical Details

### TEA Algorithm
**Tiny Encryption Algorithm (TEA)** is a block cipher:
- **Block size**: 64 bits (8 bytes)
- **Key size**: 128 bits (16 bytes)
- **Rounds**: 32 (typically)
- **Delta constant**: `0x9E3779B9` (derived from golden ratio)

### TEA Decryption Formula
For each round:
```c
v1 -= ((v0<<4) + k2) ^ (v0 + sum) ^ ((v0>>5) + k3)
v0 -= ((v1<<4) + k0) ^ (v1 + sum) ^ ((v1>>5) + k1)
sum -= delta
```

### Identifying TEA in Assembly
Look for these patterns:
- Shift left by 4 (`shl reg, 4`)
- Shift right by 5 (`shr reg, 5`)
- XOR operations between shifted values
- Loop counter of 32
- Constant `0x9E3779B9`

### Kernel Driver IOCTL
Windows kernel drivers use IOCTL (I/O Control) codes to communicate with user-mode applications:
- **IOCTL code**: `0x222004` in this challenge
- **Magic value**: `0xDEADB04B` required for authentication
- **Purpose**: Prevents casual execution, adds challenge layer

## Alternative Approaches

### Dynamic Analysis
1. Load the driver in a Windows kernel debugger (WinDbg)
2. Set breakpoint after decryption
3. Examine memory to see plaintext
4. Extract flag directly

### Automated Crypto Detection
Tools like `findcrypt` plugin for IDA can identify crypto constants like the TEA delta.

## Lessons Learned

1. **Crypto identification**: Recognizing cipher patterns in assembly
2. **TEA is simple**: Easy to implement once identified
3. **Kernel drivers**: Understanding IOCTL mechanisms
4. **Constants matter**: Crypto constants (like delta) are identifiable
5. **Static analysis works**: No need to run kernel code

## Tools Used
- **IDA Pro / Ghidra**: For disassembly and analysis
- **Python**: For implementing TEA decryption
- **struct module**: For binary data packing/unpacking

## References
- [TEA (Tiny Encryption Algorithm)](https://en.wikipedia.org/wiki/Tiny_Encryption_Algorithm)
- [Windows Kernel Drivers](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/)
- [IOCTL](https://docs.microsoft.com/en-us/windows/win32/devio/device-input-and-output-control-ioctl-)

---

**Flag**: `ZeroDays{k3rn3lm0d3b0nk}`

**Solved**: ✅

**Key Takeaway**: TEA cipher is identifiable by its characteristic shift operations (<<4, >>5) and delta constant. Kernel drivers can be analyzed statically without needing to load them, making reverse engineering safer and easier.
