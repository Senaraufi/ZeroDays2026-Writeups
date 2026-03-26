# Stego — challenge.png (draw.io Hidden Image)

## Challenge Information
- **Category**: Steganography
- **Difficulty**: Medium
- **Points**: Unknown
- **Flag**: `ZeroDays{n3v3r_g0nn4_l3t_y0u_down}`

## Description
A meme PNG (651×327px) with text: *"One does not simply... Uncrop the image to get the flag."*

The image appears to be cropped, hiding part of the content.

## Challenge Analysis

### Initial Inspection
- **Visible dimensions**: 651×327 pixels
- **File size**: Unusually large for a simple meme (~133KB)
- **Hint**: "Uncrop the image" suggests hidden content beyond visible bounds

## Solution

### Step 1: Examine PNG Chunks
Use a PNG chunk analyzer or hex editor to inspect the file structure:

```bash
pngcheck -v challenge.png
```

**Discovery**: A massive `tEXt` chunk (133,716 bytes) with keyword `mxfile`.

### Step 2: Identify draw.io Metadata
The `mxfile` keyword indicates this PNG contains an embedded **draw.io diagram**. Draw.io can save diagrams as PNG files with the diagram data embedded in metadata.

### Step 3: Extract the tEXt Chunk
```python
from PIL import Image
import struct

# Read PNG file
with open('challenge.png', 'rb') as f:
    data = f.read()

# Find tEXt chunk
idx = data.find(b'tEXt')
if idx != -1:
    # Read chunk length (4 bytes before 'tEXt')
    length = struct.unpack('>I', data[idx-4:idx])[0]
    
    # Extract chunk data
    chunk_data = data[idx+4:idx+4+length]
    
    # Split at null byte (keyword\0data)
    keyword, content = chunk_data.split(b'\x00', 1)
    print(f"Keyword: {keyword.decode()}")
    
    # Save the content
    with open('diagram.xml', 'wb') as out:
        out.write(content)
```

### Step 4: URL-Decode the Content
The draw.io XML is URL-encoded:

```python
from urllib.parse import unquote

with open('diagram.xml', 'rb') as f:
    encoded = f.read().decode('utf-8')

decoded = unquote(encoded)

with open('diagram_decoded.xml', 'w') as f:
    f.write(decoded)
```

### Step 5: Extract Embedded Image
The XML contains an `mxCell` element with an embedded JPEG in base64:

```xml
<mxCell ... image="data:image/jpeg;base64,/9j/4AAQSkZJRg..." />
```

Extract and decode the base64 JPEG:

```python
import base64
import re
from PIL import Image
import io

# Read decoded XML
with open('diagram_decoded.xml', 'r') as f:
    xml = f.read()

# Extract base64 JPEG
match = re.search(r'image="data:image/jpeg;base64,([^"]+)"', xml)
if match:
    jpeg_b64 = match.group(1)
    
    # Decode base64
    jpeg_data = base64.b64decode(jpeg_b64)
    
    # Load image
    img = Image.open(io.BytesIO(jpeg_data))
    print(f"Embedded image size: {img.size}")
    # Output: (651, 383)
    
    img.save('embedded_full.jpg')
```

### Step 6: Extract Hidden Portion
The embedded JPEG is **651×383 pixels**, but the PNG canvas is only **651×327 pixels**.

The hidden portion is the bottom **56 pixels** (383 - 327 = 56):

```python
from PIL import Image

# Load the full embedded image
full_img = Image.open('embedded_full.jpg')

# Crop the hidden portion (bottom 56 pixels)
hidden = full_img.crop((0, 327, 651, 383))

# Save and view
hidden.save('hidden_flag.png')
hidden.show()
```

The hidden portion contains the flag text: `ZeroDays{n3v3r_g0nn4_l3t_y0u_down}`

## Complete Solution Script

```python
from PIL import Image
import io
import base64
import re
import struct
from urllib.parse import unquote

# Step 1: Read PNG and extract tEXt chunk
with open('challenge.png', 'rb') as f:
    data = f.read()

# Find tEXt chunk
idx = data.find(b'tEXt')
length = struct.unpack('>I', data[idx-4:idx])[0]
chunk_data = data[idx+4:idx+4+length]
keyword, content = chunk_data.split(b'\x00', 1)

# Step 2: URL-decode the draw.io XML
xml = unquote(content.decode('utf-8'))

# Step 3: Extract base64 JPEG
match = re.search(r'image="data:image/jpeg;base64,([^"]+)"', xml)
jpeg_b64 = match.group(1)
jpeg_data = base64.b64decode(jpeg_b64)

# Step 4: Load embedded image
full_img = Image.open(io.BytesIO(jpeg_data))
print(f"Full image size: {full_img.size}")  # (651, 383)

# Step 5: Extract hidden portion
png_height = 327
full_height = full_img.size[1]
hidden_portion = full_img.crop((0, png_height, full_img.size[0], full_height))

# Save and display
hidden_portion.save('flag.png')
print(f"Hidden portion extracted: {hidden_portion.size}")
print("Flag visible in hidden_portion!")
```

## Technical Details

### PNG tEXt Chunks
PNG files can contain text metadata in `tEXt` chunks:
- **Keyword**: Identifies the type of text (e.g., "Comment", "Author", "mxfile")
- **Data**: The actual text content
- **Size**: Can be very large (this one was 133KB)

### draw.io PNG Export
draw.io (diagrams.net) can export diagrams as PNG with embedded XML:
1. The diagram is saved as compressed XML
2. XML is URL-encoded
3. Encoded XML is stored in a PNG `tEXt` chunk with keyword "mxfile"
4. Images in the diagram are embedded as base64 data URIs

### The Hidden Image Trick
The challenge used a clever technique:
1. Create a diagram with a JPEG image (651×383px)
2. Export as PNG with a smaller canvas (651×327px)
3. The PNG only shows the top portion
4. The full JPEG is still embedded in the metadata
5. Extracting the embedded image reveals the hidden bottom portion

### Why This Works
- PNG canvas size ≠ embedded image size
- draw.io preserves the full image in metadata
- The visible PNG is essentially a "crop" of the embedded image
- Metadata extraction reveals the uncropped original

## Alternative Approaches

### 1. Open in draw.io
1. Rename `challenge.png` to `challenge.drawio.png`
2. Open in draw.io
3. The full diagram with uncropped image is visible

### 2. Online PNG Metadata Viewers
Some online tools can extract and display PNG metadata chunks.

### 3. Manual Hex Editing
Search for `data:image/jpeg;base64,` in a hex editor and extract the base64 string manually.

## Lessons Learned

1. **File size matters**: Unusually large files often contain hidden data
2. **PNG metadata**: `tEXt` chunks can store arbitrary data
3. **draw.io steganography**: Diagrams embedded in PNGs are a clever hiding place
4. **Canvas vs content**: Visible dimensions don't always match embedded content
5. **Base64 in metadata**: Images can be embedded as data URIs

## Tools Used
- **Python PIL/Pillow**: For image manipulation
- **Python struct**: For parsing PNG chunks
- **urllib.parse**: For URL decoding
- **base64**: For decoding embedded images
- **Hex editor**: For initial reconnaissance

## References
- [PNG Specification](http://www.libpng.org/pub/png/spec/1.2/PNG-Contents.html)
- [draw.io](https://www.diagrams.net/)
- [Data URI Scheme](https://en.wikipedia.org/wiki/Data_URI_scheme)

---

**Flag**: `ZeroDays{n3v3r_g0nn4_l3t_y0u_down}`

**Solved**: ✅

**Key Takeaway**: PNG files can contain extensive metadata in tEXt chunks. draw.io diagrams exported as PNG preserve the full diagram data, allowing hidden content to be extracted even if the visible canvas is cropped.
