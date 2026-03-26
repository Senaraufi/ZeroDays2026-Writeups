# Stego — rare_pepe.png + pepe.exe (Pixel LSB via RE)

## Challenge Information
- **Category**: Steganography / Reverse Engineering
- **Difficulty**: Hard
- **Points**: Unknown
- **Flag**: `ZeroDays{f33ls_g00d_m4n_h1dd3n_byT3s}`

## Description
A "Rare Pepe" PNG image and a Windows x64 console executable. The binary loads the PNG and extracts the flag from specific pixels.

## Challenge Files
- `rare_pepe.png` - 227×222 pixel PNG image
- `pepe.exe` - Windows PE32+ executable

## Solution

### Step 1: Analyze the Binary
Load `pepe.exe` in a disassembler (IDA Pro, Ghidra, or Binary Ninja).

### Step 2: Identify Image Loading
The binary uses `stbi_load()` from the stb_image library:

```c
unsigned char *image_data = stbi_load("rare_pepe.png", &width, &height, &channels, 0);
```

**Parameters:**
- `filename`: "rare_pepe.png"
- `&width`: Pointer to width variable
- `&height`: Pointer to height variable
- `&channels`: Pointer to channels variable (RGB = 3)
- `desired_channels`: 0 (use image's native channels)

### Step 3: Understand the Algorithm
Reconstructed from disassembly:

```c
int width, height, channels;
unsigned char *image_data = stbi_load("rare_pepe.png", &width, &height, &channels, 0);

// width = 227, height = 222, channels = 3
int total = width * height;  // 50394 pixels

for (int i = 0; i < total; i++) {
    if (i % 7 == 0) {
        // Extract blue channel (index +2), XOR with 0x5A
        unsigned char byte = image_data[i * 3 + 2] ^ 0x5A;
        append_to_output(byte);
        
        if (byte == '}') {
            break;  // End of flag
        }
    }
}
```

**Key points:**
- Processes every 7th pixel (i % 7 == 0)
- Extracts the **blue channel** (offset +2 in RGB array)
- XORs with key `0x5A`
- Stops when `}` character is found

### Step 4: Critical Detail - stbi_load Parameter Order
**IMPORTANT:** The parameter order is:
```c
stbi_load(filename, &x, &y, &channels, desired_channels)
```
Where `x` = width, `y` = height.

This means:
- `width = 227`
- `height = 222`
- `total = 227 * 222 = 50394`

### Step 5: Implement the Extraction
```python
from PIL import Image
import numpy as np

# Load image
img = Image.open('rare_pepe.png').convert('RGB')

# Convert to flat array (matches stbi_load output)
flat = np.array(img).flatten()

# Image dimensions
width = 227
height = 222
total = width * height  # 50394

# Extract flag
result = bytearray()
for i in range(total):
    if i % 7 == 0:
        # Blue channel is at index i*3+2
        blue = int(flat[i * 3 + 2])
        
        # XOR with key
        byte = blue ^ 0x5A
        result.append(byte)
        
        # Stop at closing brace
        if byte == ord('}'):
            break

flag = bytes(result).decode()
print(flag)
# ZeroDays{f33ls_g00d_m4n_h1dd3n_byT3s}
```

## Complete Solution Script

```python
from PIL import Image
import numpy as np

# Load the image
img = Image.open('rare_pepe.png').convert('RGB')

# Convert to flat numpy array (matches stbi_load memory layout)
flat = np.array(img).flatten()

# Image dimensions (from binary analysis)
width = 227
height = 222
total = width * height  # 50394 pixels

print(f"[*] Image dimensions: {width}x{height}")
print(f"[*] Total pixels: {total}")
print(f"[*] Extracting flag from every 7th pixel...")

# Extract flag
result = bytearray()
for i in range(total):
    if i % 7 == 0:
        # Extract blue channel (RGB layout: R=0, G=1, B=2)
        blue_value = int(flat[i * 3 + 2])
        
        # XOR with key 0x5A
        decrypted_byte = blue_value ^ 0x5A
        result.append(decrypted_byte)
        
        # Stop when we hit the closing brace
        if decrypted_byte == ord('}'):
            break

# Convert to string
flag = bytes(result).decode()
print(f"\n[+] Flag: {flag}")
# ZeroDays{f33ls_g00d_m4n_h1dd3n_byT3s}
```

## Technical Details

### stbi_load Memory Layout
The `stbi_load()` function returns a flat array of pixels:
```
[R0, G0, B0, R1, G1, B1, R2, G2, B2, ...]
```

For pixel `i`:
- Red channel: `data[i * 3 + 0]`
- Green channel: `data[i * 3 + 1]`
- Blue channel: `data[i * 3 + 2]`

### Pixel Selection Pattern
Every 7th pixel is used:
- Pixel 0, 7, 14, 21, 28, ...
- Total flag bytes: ~7,200 / 7 ≈ 1,028 pixels checked
- Actual flag length: ~37 bytes (stops at `}`)

### XOR Key
- **Key**: `0x5A` (90 in decimal)
- **Binary**: `01011010`
- **Purpose**: Simple obfuscation of the flag data

### Why Blue Channel?
The choice of blue channel is arbitrary - could have been red or green. The binary specifically uses offset `+2` which corresponds to blue in RGB order.

### Common Pitfall
**Wrong parameter order assumption:**
If you assume `stbi_load(filename, &y, &x, ...)` (height, width), you get:
- `width = 222, height = 227`
- `total = 222 * 227 = 50394` (same number!)
- But pixel indexing is wrong, producing garbage

The correct order is `(filename, &x, &y, ...)` where x=width, y=height.

## Alternative Approaches

### 1. Dynamic Analysis
Run the executable in a debugger:
- Set breakpoint after flag extraction
- Examine memory to see the flag
- No need to understand the algorithm

### 2. Automated Stego Detection
Tools like `zsteg` or `stegsolve` might detect the pattern, though the XOR and specific pixel selection make this unlikely.

### 3. Brute Force XOR Keys
If the XOR key was unknown, try all 256 possible keys and check for readable text.

## Lessons Learned

1. **Reverse engineering is essential**: Understanding the binary reveals the exact algorithm
2. **Parameter order matters**: API function parameter order must be correct
3. **LSB alternatives**: Not all pixel stego uses LSB - this uses full channel values
4. **XOR obfuscation**: Simple XOR can hide data effectively
5. **Pattern-based extraction**: Every Nth pixel is a common pattern

## Tools Used
- **IDA Pro / Ghidra**: For binary reverse engineering
- **Python PIL/Pillow**: For image loading
- **NumPy**: For efficient array operations
- **Hex editor**: For examining the binary

## References
- [stb_image Library](https://github.com/nothings/stb)
- [LSB Steganography](https://en.wikipedia.org/wiki/Steganography#Digital)
- [XOR Cipher](https://en.wikipedia.org/wiki/XOR_cipher)

---

**Flag**: `ZeroDays{f33ls_g00d_m4n_h1dd3n_byT3s}`

**Solved**: ✅

**Key Takeaway**: Steganography challenges can require reverse engineering to understand the extraction algorithm. Pay careful attention to API function parameter orders and data structure layouts when reimplementing binary algorithms.
