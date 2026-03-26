# Rev — karen (ELF XOR Decryption)

## Challenge Information
- **Category**: Reverse Engineering
- **Difficulty**: Medium
- **Points**: Unknown
- **Flag**: `ZeroDays{d0_u_kn0w_wh0_1_@m?!}`

## Description
A 64-bit Linux ELF binary named `karen` with the message: *"I demand to speak to the root user IMMEDIATELY!"*

Running the binary prints a mangled string that needs to be decrypted.

## Challenge Analysis

### Initial Execution
```bash
$ ./karen
# Outputs mangled text
```

The output is clearly encrypted or obfuscated text.

## Solution

### Step 1: Static Analysis
Using a disassembler (Ghidra, IDA, or Binary Ninja), we can identify:

1. **Secret string** in `.rodata` section:
   ```
   Tk|aJow}pj;T{Te`;yTyf;T:TKc4*v
   ```

2. **Bitmap array** (`bitmap_num`):
   ```c
   int bitmap_num[] = {8, 10, 11, 13, 16, 18, 21, 22, 23, 24, 25, 27, 28, 29, 30};
   ```

3. **Transform function** that performs XOR operations

### Step 2: Reverse the Algorithm
The `transform()` function logic:

```c
for (int i = 0; i < len(secret); i++) {
    if (i in bitmap_num) {
        output[i] = secret[i] ^ 0x0b;
    } else {
        output[i] = secret[i] ^ 0x0e;
    }
}
```

**Key insight**: Each byte is XORed with either `0x0b` or `0x0e` depending on whether its index is in the `bitmap_num` array.

### Step 3: Implement the Decryption
```python
secret = b'Tk|aJow}pj;T{Te`;yTyf;T:TKc4*v'
bitmap_num = {8, 10, 11, 13, 16, 18, 21, 22, 23, 24, 25, 27, 28, 29, 30}

result = []
for i in range(len(secret)):
    if i in bitmap_num:
        key = 0x0b
    else:
        key = 0x0e
    result.append(secret[i] ^ key)

flag = bytes(result)
print(flag.decode())
# ZeroDays{d0_u_kn0w_wh0_1_@m?!}
```

## Complete Solution Script

```python
# Extracted from binary analysis
secret = b'Tk|aJow}pj;T{Te`;yTyf;T:TKc4*v'
bitmap_num = {8, 10, 11, 13, 16, 18, 21, 22, 23, 24, 25, 27, 28, 29, 30}

# XOR decryption
result = []
for i in range(len(secret)):
    # Choose XOR key based on index
    key = 0x0b if i in bitmap_num else 0x0e
    result.append(secret[i] ^ key)

# Convert to string
flag = bytes(result).decode()
print(flag)
# ZeroDays{d0_u_kn0w_wh0_1_@m?!}
```

## Technical Details

### XOR Cipher
XOR (exclusive OR) is a common encryption method in CTF challenges:
- **Property**: `A ^ B ^ B = A` (XOR is its own inverse)
- **Usage**: Simple obfuscation of strings in binaries
- **Weakness**: Easy to reverse if the key is known

### Bitmap-Based Key Selection
The algorithm uses a bitmap to select between two XOR keys:
- **Indices in bitmap**: XOR with `0x0b`
- **Indices not in bitmap**: XOR with `0x0e`

This creates a simple polyalphabetic substitution cipher.

### Static Analysis Workflow
1. **Load binary** in disassembler (Ghidra/IDA)
2. **Find strings** in `.rodata` section
3. **Identify functions** that reference the strings
4. **Analyze algorithm** by reading assembly/decompiled code
5. **Extract constants** (XOR keys, bitmap array)
6. **Implement decryption** in Python

## Alternative Approaches

### Dynamic Analysis
1. Run the binary in a debugger (GDB)
2. Set breakpoint after the transform function
3. Examine memory to see the decrypted string
4. Extract the flag directly

### Automated String Extraction
```bash
# Sometimes simple tools work
strings karen | grep -i zero
```

## Lessons Learned

1. **Static analysis is powerful**: Disassemblers reveal algorithm logic
2. **XOR is common**: Many binaries use XOR for string obfuscation
3. **Look for patterns**: Bitmap arrays and constant keys are clues
4. **Multiple approaches**: Both static and dynamic analysis work
5. **Extract constants carefully**: Bitmap indices must be exact

## Tools Used
- **Ghidra / IDA Pro**: For disassembly and decompilation
- **Python**: For implementing the decryption algorithm
- **GDB**: Optional for dynamic analysis

## References
- [XOR Cipher](https://en.wikipedia.org/wiki/XOR_cipher)
- [Ghidra](https://ghidra-sre.org/)
- [ELF Binary Format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)

---

**Flag**: `ZeroDays{d0_u_kn0w_wh0_1_@m?!}`

**Solved**: ✅

**Key Takeaway**: XOR encryption in binaries is easily reversed through static analysis. Look for constant keys and bitmap/index arrays that control the encryption logic.
