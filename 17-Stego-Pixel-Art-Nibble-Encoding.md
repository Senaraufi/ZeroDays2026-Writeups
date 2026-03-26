# Stego — Pixel Art Nibble Encoding

## Challenge Information
- **Category**: Steganography
- **Difficulty**: Medium
- **Points**: Unknown
- **Flag**: `ZeroDays{p1xel_4rt-is-th3_b3st-steg0-m3thod!}`

## Description
A steganography challenge where pixels encode data using a 16-color palette. The hint: *"The pixels are talking to me. They have an important message. 6 colors = 4 bits per pixel. Classic!"*

## Challenge Analysis

### The Encoding Scheme
- **16-color palette** = 4 bits per color (2^4 = 16)
- Each pixel in the challenge image maps to one of 16 colors
- Each color represents a **nibble** (4 bits, values 0-15)
- Two pixels = 2 nibbles = 1 byte

### Files Provided
1. **Hint image**: 16-color palette reference
2. **Challenge image**: Encoded data as colored pixels

## Solution

### Step 1: Map Pixels to Nibbles

Each pixel color in the challenge image corresponds to a nibble value (0-15) based on the 16-color palette from the hint image.

### Step 2: Decode Nibbles to Bytes

```python
from PIL import Image

# Load the challenge image
img = Image.open('challenge.png')
pixels = list(img.getdata())

# Load the palette from hint image
hint = Image.open('hint.png')
palette_colors = list(hint.getdata())[:16]  # First 16 colors

# Create color-to-nibble mapping
color_to_nibble = {color: i for i, color in enumerate(palette_colors)}

# Decode pixels to nibbles
nibbles = [color_to_nibble[pixel] for pixel in pixels]

# Pair nibbles to form bytes
data = bytes((nibbles[i] << 4) | nibbles[i+1] for i in range(0, len(nibbles), 2))

print(data[:100])  # Check first 100 bytes
```

### Step 3: Identify File Type

The decoded data starts with `PK\x03\x04` — the magic bytes for a **ZIP archive**!

```python
# Check magic bytes
print(data[:4])  # b'PK\x03\x04'
```

### Step 4: Extract ZIP Archive

```python
# Save as ZIP file
with open('decoded.zip', 'wb') as f:
    f.write(data)

# Extract contents
import zipfile
with zipfile.ZipFile('decoded.zip', 'r') as zip_ref:
    zip_ref.extractall('extracted')
    print(zip_ref.namelist())
```

### Step 5: Find the Flag

The ZIP contains a LibreOffice ODT document. Opening it reveals the flag:

```
ZeroDays{p1xel_4rt-is-th3_b3st-steg0-m3thod!}
```

## Complete Solution Script

```python
from PIL import Image
import zipfile

# Load images
challenge = Image.open('challenge.png')
hint = Image.open('hint.png')

# Get pixel data
challenge_pixels = list(challenge.getdata())
palette_colors = list(hint.getdata())[:16]

# Create color-to-nibble mapping
color_to_nibble = {color: i for i, color in enumerate(palette_colors)}

# Decode pixels to nibbles
nibbles = [color_to_nibble[pixel] for pixel in challenge_pixels]

# Convert nibble pairs to bytes
data = bytes((nibbles[i] << 4) | nibbles[i+1] for i in range(0, len(nibbles), 2))

# Save as ZIP
with open('decoded.zip', 'wb') as f:
    f.write(data)

# Extract ZIP
with zipfile.ZipFile('decoded.zip', 'r') as zip_ref:
    zip_ref.extractall('extracted')
    print(f"Extracted files: {zip_ref.namelist()}")

# The ODT document contains the flag
print("\nFlag: ZeroDays{p1xel_4rt-is-th3_b3st-steg0-m3thod!}")
```

## Technical Details

### Nibble Encoding
- **Nibble**: 4 bits, representing values 0-15
- **Byte**: 2 nibbles (8 bits)
- High nibble (first pixel) is shifted left 4 bits
- Low nibble (second pixel) fills the lower 4 bits

### ZIP File Structure
The decoded data forms a valid ZIP archive containing:
- LibreOffice ODT (Open Document Text) file
- ODT is essentially a ZIP containing XML and resources
- Flag was embedded in the document content

### Why This Works
1. 16 colors provide exactly 4 bits of information per pixel
2. Compact encoding: 2 pixels = 1 byte
3. Can encode arbitrary binary data (not just text)
4. Visually looks like abstract pixel art

## Prevention

For CTF organizers wanting to create similar challenges:
- Use palette-based images (PNG with indexed colors)
- Ensure pixel order is preserved
- Consider adding noise or decoy pixels
- Can nest multiple layers (ZIP → ODT → embedded content)

## Lessons Learned

1. **Palette Analysis**: Limited color palettes often indicate data encoding
2. **Nibble Encoding**: 16 colors = 4 bits is a classic steganography technique
3. **File Signatures**: Always check magic bytes to identify file types
4. **Nested Formats**: ZIP archives can contain other structured formats
5. **Pixel Order Matters**: Sequential pixel reading is crucial for decoding

## Tools Used

- **Python PIL/Pillow**: Image processing and pixel extraction
- **Python zipfile**: ZIP archive handling
- **LibreOffice**: ODT document viewing
- **Hex editor**: Verifying file signatures

## References

- [PNG File Format](https://en.wikipedia.org/wiki/Portable_Network_Graphics)
- [ZIP File Format](https://en.wikipedia.org/wiki/ZIP_(file_format))
- [ODT Format](https://en.wikipedia.org/wiki/OpenDocument)
- [Nibble Encoding](https://en.wikipedia.org/wiki/Nibble)

---

**Flag**: `ZeroDays{p1xel_4rt-is-th3_b3st-steg0-m3thod!}`

**Solved**: ✅
