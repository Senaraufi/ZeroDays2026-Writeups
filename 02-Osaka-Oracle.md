# Osaka's Oracle - Padding Oracle Attack

## Challenge Information
- **Category**: Cryptography
- **Difficulty**: Hard
- **Points**: 
- **Flag**: `ZeroDays{https://youtu.be/pquWcqqd_b4}`
- **Service**: `https://osakasoracle.zerodays.events`

## Description
A cryptography challenge involving a padding oracle vulnerability in an AES-CBC encryption service. The service leaks information about padding validity through different error messages.

## Vulnerability Analysis

The service implements AES-CBC encryption with PKCS#7 padding but reveals whether the padding is valid through distinct error messages:
- **Valid padding**: "america ya"
- **Invalid padding**: "oh my gah"

This information leakage creates a classic **padding oracle vulnerability** that allows an attacker to decrypt ciphertext without knowing the encryption key.

## Padding Oracle Attack Theory

### How AES-CBC Works
```
Plaintext:  [Block 1] [Block 2] [Block 3]
            ↓          ↓          ↓
Key:        [  AES  ] [  AES  ] [  AES  ]
            ↓          ↓          ↓
Ciphertext: [Block 1] [Block 2] [Block 3]

Each block is XORed with the previous ciphertext block (CBC mode)
```

### PKCS#7 Padding
- Pads data to block size (16 bytes for AES)
- Padding byte value = number of padding bytes
- Examples:
  - 1 byte needed: `0x01`
  - 2 bytes needed: `0x02 0x02`
  - 16 bytes needed: `0x10 0x10 ... 0x10`

### The Oracle
When we modify ciphertext and send it to the service:
1. Service decrypts the ciphertext
2. Service checks if padding is valid
3. Service responds with different messages based on validity
4. We can use this to deduce plaintext bytes

### Attack Mechanism
For each byte of plaintext:
1. Modify the previous ciphertext block
2. Send modified ciphertext to oracle
3. Check if padding is valid
4. Use response to deduce plaintext byte
5. Repeat for all bytes in all blocks

## Solution

### Step 1: Identify the Oracle
```python
import requests

def check_padding(ciphertext):
    response = requests.post('https://osakasoracle.zerodays.events', 
                            data={'ciphertext': ciphertext})
    
    if "america ya" in response.text:
        return True  # Valid padding
    elif "oh my gah" in response.text:
        return False  # Invalid padding
    
    return None
```

### Step 2: Implement Padding Oracle Attack
```python
def decrypt_block(prev_block, curr_block, oracle):
    """Decrypt a single block using padding oracle"""
    plaintext = bytearray(16)
    
    # Work backwards from last byte to first
    for pad_value in range(1, 17):
        # For each byte position
        for guess in range(256):
            # Craft modified ciphertext
            modified = bytearray(prev_block)
            
            # Set known bytes to produce correct padding
            for i in range(16 - pad_value + 1, 16):
                modified[i] = prev_block[i] ^ plaintext[i] ^ pad_value
            
            # Try current guess
            modified[16 - pad_value] = prev_block[16 - pad_value] ^ guess ^ pad_value
            
            # Check with oracle
            if oracle(bytes(modified) + curr_block):
                plaintext[16 - pad_value] = guess
                break
    
    return bytes(plaintext)
```

### Step 3: Decrypt All Blocks
```python
def padding_oracle_decrypt(ciphertext, oracle):
    # Split into 16-byte blocks
    blocks = [ciphertext[i:i+16] for i in range(0, len(ciphertext), 16)]
    
    plaintext = b''
    
    # Decrypt each block (skip IV/first block)
    for i in range(1, len(blocks)):
        decrypted = decrypt_block(blocks[i-1], blocks[i], oracle)
        plaintext += decrypted
    
    # Remove PKCS#7 padding
    padding_len = plaintext[-1]
    return plaintext[:-padding_len]
```

### Step 4: Extract the Flag
```python
# Run the attack
decrypted = padding_oracle_decrypt(ciphertext, check_padding)

print(f"Decrypted: {decrypted.decode()}")
# Output: sata andagi sata andagi ZeroDays{https://youtu.be/pquWcqqd_b4} s...

# Extract flag
import re
flag = re.search(r'ZeroDays\{[^}]+\}', decrypted.decode())
print(f"Flag: {flag.group()}")
```

## Technical Details

### Decrypted Plaintext
```
sata andagi sata andagi ZeroDays{https://youtu.be/pquWcqqd_b4} s...
```

The plaintext contains:
- Repeated phrase: "sata andagi" (a type of Okinawan donut)
- The flag: `ZeroDays{https://youtu.be/pquWcqqd_b4}`
- Additional padding/text

### Oracle Responses
- **"america ya"**: Valid PKCS#7 padding detected
- **"oh my gah"**: Invalid padding detected

### Attack Complexity
- **Requests per byte**: ~128 on average (256 worst case)
- **Total requests**: ~128 × 16 × number_of_blocks
- **Time complexity**: O(n × 256) where n is ciphertext length

## Optimization Techniques

### 1. Parallel Processing
```python
from concurrent.futures import ThreadPoolExecutor

def decrypt_parallel(blocks, oracle):
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [executor.submit(decrypt_block, blocks[i-1], blocks[i], oracle) 
                   for i in range(1, len(blocks))]
        return b''.join(f.result() for f in futures)
```

### 2. Smart Guessing
```python
# Try common bytes first (space, letters, etc.)
common_bytes = [0x20, 0x65, 0x61, 0x6F, 0x69, 0x6E, 0x73, 0x74]
for guess in common_bytes + list(range(256)):
    # ... oracle check
```

### 3. Caching
```python
cache = {}

def cached_oracle(ciphertext):
    if ciphertext in cache:
        return cache[ciphertext]
    result = oracle(ciphertext)
    cache[ciphertext] = result
    return result
```

## Prevention

### 1. Use Authenticated Encryption
```python
# Instead of AES-CBC, use AES-GCM
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

aesgcm = AESGCM(key)
ciphertext = aesgcm.encrypt(nonce, plaintext, associated_data)
```

### 2. Constant-Time Padding Validation
```python
def constant_time_compare(a, b):
    """Compare two byte strings in constant time"""
    if len(a) != len(b):
        return False
    result = 0
    for x, y in zip(a, b):
        result |= x ^ y
    return result == 0
```

### 3. Generic Error Messages
```python
# BAD: Reveals padding information
if not valid_padding(plaintext):
    return "Invalid padding"
else:
    return "Success"

# GOOD: Generic error message
try:
    plaintext = decrypt_and_verify(ciphertext)
    return "Success"
except:
    return "Decryption failed"  # Same message for all errors
```

### 4. Use Modern Cryptography Libraries
- **Don't**: Implement your own crypto
- **Do**: Use well-tested libraries (libsodium, cryptography.io)
- **Do**: Use AEAD modes (GCM, ChaCha20-Poly1305)

## Tools Used
- **Python**: For scripting the attack
- **requests**: For HTTP communication
- **Custom padding oracle implementation**: For the attack

## Scripts
- `gigachad/fast_padding_oracle.py` - Optimized attack implementation
- `gigachad/padding_oracle_attack.py` - Basic attack implementation
- `gigachad/padding_oracle_v2.py` - Alternative implementation

## Lessons Learned

1. **Information leakage is dangerous**: Even small differences in error messages can be exploited
2. **Padding oracles are practical**: Not just theoretical attacks
3. **Timing matters**: Even timing differences can create oracles
4. **Use authenticated encryption**: AEAD modes prevent these attacks
5. **Constant-time operations**: Critical for cryptographic implementations

## References
- [Padding Oracle Attack Explained](https://en.wikipedia.org/wiki/Padding_oracle_attack)
- [CBC Padding Oracle Attacks](https://robertheaton.com/2013/07/29/padding-oracle-attack/)
- CWE-354: Improper Validation of Integrity Check Value

---

**Flag**: `ZeroDays{https://youtu.be/pquWcqqd_b4}`

**Solved**: ✅

**Key Takeaway**: Never reveal information about padding validity. Use authenticated encryption (AEAD) to prevent padding oracle attacks entirely.
