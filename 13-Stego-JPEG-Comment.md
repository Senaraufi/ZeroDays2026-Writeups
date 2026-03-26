# Stego — challenge_2.jpg (JPEG Comment Metadata)

## Challenge Information
- **Category**: Steganography
- **Difficulty**: Easy
- **Points**: Unknown
- **Flag**: `ZeroDays{ex1f-d4ta-h0ldz-many-s3rets!}`

## Description
A SpongeBob "to-do list" meme with the hint: *"fairly meta"* → suggesting metadata.

## Challenge Analysis

The hint "fairly meta" strongly suggests the flag is hidden in the image's metadata rather than in the pixel data itself.

## Solution

### Step 1: Check EXIF/Metadata
Use any metadata viewer to examine the JPEG:

```bash
exiftool challenge_2_.jpg
```

Or use Python:

```python
from PIL import Image

img = Image.open('challenge_2_.jpg')
print(img.info)
```

### Step 2: Find the Comment Field
The JPEG file has a **comment field** containing a base64-encoded string:

```python
from PIL import Image

img = Image.open('challenge_2_.jpg')
comment = img.info.get('comment')
print(comment)
# b'WmVyb0RheXN7ZXgxZi1kNHRhLWgwbGR6LW1hbnktczNyZXRzIX0='
```

### Step 3: Decode Base64
```python
import base64

comment = b'WmVyb0RheXN7ZXgxZi1kNHRhLWgwbGR6LW1hbnktczNyZXRzIX0='
flag = base64.b64decode(comment).decode()
print(flag)
# ZeroDays{ex1f-d4ta-h0ldz-many-s3rets!}
```

## Complete Solution Script

```python
from PIL import Image
import base64

# Open the image
img = Image.open('challenge_2_.jpg')

# Extract comment from metadata
comment = img.info.get('comment')
print(f"Comment (base64): {comment}")

# Decode base64 to get flag
flag = base64.b64decode(comment).decode()
print(f"\nFlag: {flag}")
# ZeroDays{ex1f-d4ta-h0ldz-many-s3rets!}
```

## Technical Details

### JPEG Comment Field
JPEG files can contain a **COM (Comment) marker** that stores arbitrary text:
- **Marker**: `0xFFFE`
- **Purpose**: Store image comments or descriptions
- **Size**: Variable length
- **Common use**: Copyright info, camera settings, or in this case, flags

### EXIF vs Comment
- **EXIF**: Structured metadata (camera, GPS, timestamps)
- **Comment**: Freeform text field
- **Both**: Can be viewed with metadata tools

### Why Metadata?
The hint "fairly meta" is a play on words:
- **Meta** = metadata
- **Fair** = straightforward, not deeply hidden

## Alternative Approaches

### 1. Command-Line Tools
```bash
# Using exiftool
exiftool challenge_2_.jpg | grep -i comment

# Using strings
strings challenge_2_.jpg | grep -i zero

# Using identify (ImageMagick)
identify -verbose challenge_2_.jpg | grep -i comment
```

### 2. Online Metadata Viewers
Upload to any online EXIF viewer:
- Jeffrey's Image Metadata Viewer
- Metapicz
- Online EXIF Viewer

### 3. Hex Editor
Search for the base64 string in a hex editor - it's stored in plain text after the COM marker.

## Common JPEG Metadata Fields

| Field | Description | Common Use |
|-------|-------------|------------|
| Comment | Freeform text | Descriptions, notes |
| Artist | Creator name | Copyright info |
| Copyright | Copyright notice | Legal info |
| DateTime | Creation time | Timestamp |
| Make/Model | Camera info | Device identification |
| GPS | Location data | Geolocation |
| UserComment | User notes | Additional info |

## Lessons Learned

1. **Check metadata first**: "Meta" hints often mean metadata
2. **JPEG comments**: Easy hiding place for flags
3. **Base64 is common**: Metadata often contains base64 strings
4. **Multiple tools**: Use exiftool, PIL, or online viewers
5. **Low-hanging fruit**: Not all stego is complex pixel manipulation

## Tools Used
- **Python PIL/Pillow**: For reading image metadata
- **exiftool**: Command-line metadata viewer
- **base64**: For decoding the flag

## References
- [JPEG File Format](https://en.wikipedia.org/wiki/JPEG#Syntax_and_structure)
- [EXIF Metadata](https://en.wikipedia.org/wiki/Exif)
- [ExifTool](https://exiftool.org/)

---

**Flag**: `ZeroDays{ex1f-d4ta-h0ldz-many-s3rets!}`

**Solved**: ✅

**Key Takeaway**: When a challenge hints at "meta" or metadata, check the image's EXIF/comment fields first. JPEG comment fields are an easy and common place to hide flags in steganography challenges.
