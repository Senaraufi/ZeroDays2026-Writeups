# Warmup/Encoding — U WOT M8? (Multi-layer Base64)

## Challenge Information
- **Category**: Warmup / Encoding
- **Difficulty**: Easy
- **Points**: Unknown
- **Flag**: `ZeroDays{we_hate_the_clankers}`

## Description
A multi-layered encoding challenge with the hint: *"Just don't forget to go backwards."*

## Challenge Data

Given encoded string:
```
PVEyTnpjak0zVWpOaVpUWjJFak5qWnpNMllXTjFZRE8yUXpObVZUTjJRek54WURPMllXTjFZek4zSTJOemNUTzNFak4wUWpaMkl6TjFZVFkxb1RSRDVVUVVOVlRWTmtVSk5FSVo1VVFnSVZSRTVVVmdRVlNnVTBRT1ZrVUZaVVJTQmlVRlpWUk9CQ1p1...
```

## Solution

The challenge involves multiple layers of encoding that must be reversed in the correct order.

### Step 1: First Base64 Decode
```python
import base64

encoded = 'PVEyTnpjak0z...'  # full string
decoded1 = base64.b64decode(encoded).decode()
```

### Step 2: Reverse the String
The hint "don't forget to go backwards" indicates we need to reverse the decoded string:
```python
reversed_str = decoded1[::-1]
```

### Step 3: Second Base64 Decode
```python
decoded2 = base64.b64decode(reversed_str).decode()
```

This reveals a message containing a hex string with the label "NEVER REFERENCE IT" or similar misdirection.

### Step 4: Extract and Decode the Hex String
The message contains a hex string that the challenge author tries to hide with reverse psychology:
```python
hex_string = '5a65726f446179737b77655f686174655f7468655f636c616e6b6572737d'
flag = bytes.fromhex(hex_string).decode()
print(flag)
# ZeroDays{we_hate_the_clankers}
```

## Complete Solution Script

```python
import base64

# Given encoded string
s = 'PVEyTnpjak0zVWpOaVpUWjJFak5qWnpNMllXTjFZRE8yUXpObVZUTjJRek54WURPMllXTjFZek4zSTJOemNUTzNFak4wUWpaMkl6TjFZVFkxb1RSRDVVUVVOVlRWTmtVSk5FSVo1VVFnSVZSRTVVVmdRVlNnVTBRT1ZrVUZaVVJTQmlVRlpWUk9CQ1p1...'

# Step 1: Base64 decode
decoded = base64.b64decode(s).decode()

# Step 2: Reverse the string
rev = decoded[::-1]

# Step 3: Base64 decode again
decoded2 = base64.b64decode(rev).decode()

# Step 4: Extract hex string from message
# The message says something like "ignore this hash: 5a65726f446179737b77655f686174655f7468655f636c616e6b6572737d"
hex_string = '5a65726f446179737b77655f686174655f7468655f636c616e6b6572737d'

# Step 5: Decode hex to get flag
flag = bytes.fromhex(hex_string).decode()
print(flag)
# ZeroDays{we_hate_the_clankers}
```

## Technical Details

### Encoding Layers
1. **Base64 encoding** (outer layer)
2. **String reversal** (middle layer)
3. **Base64 encoding** (inner layer)
4. **Hex encoding** (flag itself)

### Decoding Process
The decoding must be done in reverse order:
1. Base64 decode → get reversed string
2. Reverse string → get base64 string
3. Base64 decode → get message with hex
4. Hex decode → get flag

### The Misdirection
The challenge author uses reverse psychology by labeling the hex string as something to "NEVER REFERENCE" or "ignore this hash". This is a classic CTF trick - the thing you're told to ignore is usually the answer.

## Lessons Learned

1. **Multi-layer encoding**: Challenges can stack multiple encoding schemes
2. **Hints are important**: "go backwards" literally meant string reversal
3. **Reverse psychology**: Things labeled "don't look here" are often the solution
4. **Systematic approach**: Try each decoding step methodically
5. **Base64 is common**: Base64 appears frequently in CTF challenges

## Common Encoding Types in CTFs

- **Base64**: Most common, recognizable by `=` padding and alphanumeric characters
- **Hex**: Pairs of hexadecimal digits (0-9, a-f)
- **ROT13**: Caesar cipher with 13-character rotation
- **URL encoding**: Uses `%XX` format
- **Binary**: Strings of 0s and 1s

## Tools Used
- **Python**: Built-in `base64` module
- **String manipulation**: Slicing with `[::-1]` for reversal
- **Hex decoding**: `bytes.fromhex()`

## References
- [Base64 Encoding](https://en.wikipedia.org/wiki/Base64)
- [String Reversal in Python](https://www.geeksforgeeks.org/reverse-string-python-5-different-ways/)

---

**Flag**: `ZeroDays{we_hate_the_clankers}`

**Solved**: ✅

**Key Takeaway**: CTF challenges often use misdirection and reverse psychology. The thing you're told to ignore is frequently the answer. Always follow hints literally - "go backwards" meant actual string reversal.
