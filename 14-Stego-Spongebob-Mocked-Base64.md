# Stego — spongebob.jpg (Mocked Base64 Casing)

## Challenge Information
- **Category**: Steganography
- **Difficulty**: Medium
- **Points**: Unknown
- **Flag**: `zerodays{bawkbwakbkawkbawkbawk}`

## Description
A Mocking SpongeBob meme with an encoded string and two hints:
- MD5 hash of flag: `703799b0f0c297e9b629c847235d886f`
- *"look at the flag carefully"* → lowercase flag format

## Challenge Data

**Encoded string:**
```
EmVYb2rHexN7ymf3a2J3yWtIa2f3A2JhD2tIyXdrFQ==
```

## Challenge Analysis

The Mocking SpongeBob meme randomly alternates between uppercase and lowercase letters, mimicking the mocking speech pattern. This is the key to the challenge.

### The Problem
The encoded string appears to be base64, but decoding it produces garbage because the **case of letters has been randomized**.

## Solution

### Step 1: Recognize the Pattern
The meme theme suggests the base64 string has been "mocked" - letters have random casing that breaks normal base64 decoding.

### Step 2: Identify Known Prefix
The hint "look at the flag carefully" suggests the flag format is **lowercase**: `zerodays{` (not `ZeroDays{`)

Base64 encoding of `zerodays{`:
```python
import base64
print(base64.b64encode(b'zerodays{').decode())
# emVyb2RheXN7
```

This gives us the correct casing for the first 12 characters.

### Step 3: Constrain the Search Space
- Total base64 string length: 36 characters
- Known prefix: 12 characters (`emVyb2RheXN7`)
- Remaining: 24 characters
- Alphabetic characters in remaining: ~23 positions
- Search space: 2²³ ≈ 8.4 million combinations

### Step 4: Brute Force with MD5 Verification
Try all case combinations of the alphabetic characters, verifying each with the MD5 hash:

```python
import base64
import hashlib
import itertools

encoded = 'EmVYb2rHexN7ymf3a2J3yWtIa2f3A2JhD2tIyXdrFQ=='
target_md5 = '703799b0f0c297e9b629c847235d886f'

# Known prefix from 'zerodays{' = 'emVyb2RheXN7'
fixed_prefix = 'emVyb2RheXN7'
rest = encoded[12:]

# Find all alphabetic positions in the remaining string
alpha_positions = [i for i, c in enumerate(rest) if c.isalpha()]

# Try all case combinations
for combo in itertools.product(*[[c.upper(), c.lower()] for c in [rest[i] for i in alpha_positions]]):
    # Build candidate string
    candidate = list(rest)
    for bit, pos in zip(combo, alpha_positions):
        candidate[pos] = bit
    
    # Combine with known prefix
    full = fixed_prefix + ''.join(candidate)
    
    try:
        # Decode base64
        decoded = base64.b64decode(full)
        
        # Check MD5
        if hashlib.md5(decoded).hexdigest() == target_md5:
            print(f"Flag: {decoded.decode()}")
            break
    except:
        continue
```

## Complete Solution Script

```python
import base64
import hashlib
import itertools

# Given data
encoded = 'EmVYb2rHexN7ymf3a2J3yWtIa2f3A2JhD2tIyXdrFQ=='
target_md5 = '703799b0f0c297e9b629c847235d886f'

# Known prefix (lowercase 'zerodays{')
fixed_prefix = 'emVyb2RheXN7'
rest = encoded[12:]

# Find alphabetic character positions
alpha_idx = [i for i, c in enumerate(rest) if c.isalpha()]

print(f"[*] Brute forcing {len(alpha_idx)} alphabetic positions...")
print(f"[*] Search space: 2^{len(alpha_idx)} = {2**len(alpha_idx):,} combinations")

# Try all case combinations
for i, combo in enumerate(itertools.product(*[[c.upper(), c.lower()] for c in [rest[j] for j in alpha_idx]])):
    # Build candidate
    candidate = list(rest)
    for bit, pos in zip(combo, alpha_idx):
        candidate[pos] = bit
    
    full = fixed_prefix + ''.join(candidate)
    
    try:
        decoded = base64.b64decode(full)
        
        # Verify with MD5
        if hashlib.md5(decoded).hexdigest() == target_md5:
            print(f"\n[+] Found after {i+1:,} attempts!")
            print(f"[+] Flag: {decoded.decode()}")
            # zerodays{bawkbwakbkawkbawkbawk}
            break
    except:
        pass
    
    if i % 100000 == 0 and i > 0:
        print(f"[*] Tried {i:,} combinations...")
```

## Technical Details

### Mocking SpongeBob Meme
The meme format alternates letter casing to mock someone:
- Normal: "one does not simply"
- Mocked: "OnE dOeS nOt SiMpLy"

This challenge applies the same concept to base64 encoding.

### Base64 Case Sensitivity
Base64 encoding is **case-sensitive**:
- `A` ≠ `a` in base64
- Changing case produces different decoded values
- Random casing breaks the encoding

### The Flag Content
`bawkbwakbkawkbawkbawk` - The SpongeBob chicken sound, perfectly fitting the meme theme.

### Optimization Techniques
1. **Known prefix**: Reduces search space significantly
2. **MD5 verification**: Fast hash check before full validation
3. **Early termination**: Stop as soon as match is found
4. **Itertools.product**: Efficient combination generation

### Why This Works
- We know the first 12 characters (from `zerodays{`)
- Only need to brute force the remaining ~23 alphabetic positions
- 2²³ is computationally feasible (< 10 million)
- MD5 hash provides quick verification

## Alternative Approaches

### 1. Pattern Recognition
If you notice patterns in the original casing, you might guess the correct combination faster.

### 2. Dictionary Attack
If the flag contains dictionary words, try common word combinations.

### 3. Partial Decoding
Decode portions of the string to narrow down possibilities.

## Lessons Learned

1. **Meme themes matter**: The mocking meme hints at case randomization
2. **Known plaintext attacks**: Knowing part of the plaintext helps
3. **Brute force is viable**: 2²³ combinations is manageable
4. **MD5 for verification**: Hash checks are fast and reliable
5. **Case sensitivity**: Base64 casing must be exact

## Performance Notes
- **Search space**: ~8.4 million combinations
- **Runtime**: 1-5 minutes on modern hardware
- **Optimization**: Could parallelize for faster results

## Tools Used
- **Python itertools**: For generating combinations
- **hashlib**: For MD5 verification
- **base64**: For decoding attempts

## References
- [Mocking SpongeBob Meme](https://knowyourmeme.com/memes/mocking-spongebob)
- [Base64 Encoding](https://en.wikipedia.org/wiki/Base64)
- [Brute Force Attack](https://en.wikipedia.org/wiki/Brute-force_attack)

---

**Flag**: `zerodays{bawkbwakbkawkbawkbawk}`

**Solved**: ✅

**Key Takeaway**: Meme themes in challenges often hint at the solution method. The Mocking SpongeBob meme's characteristic random casing was applied to base64 encoding. Known plaintext (the flag prefix) combined with brute forcing makes this attack feasible.
