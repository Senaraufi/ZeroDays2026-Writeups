# Warmup/Rev — r1ckr0ll.ps1 (PowerShell Stego)

## Challenge Information
- **Category**: Warmup / Reverse Engineering
- **Difficulty**: Easy
- **Points**: Unknown
- **Flag**: `ZeroDays{n3v3r_g0nn4_l3t_y0u_down}`

## Description
A PowerShell script that spins up a local HTTP server serving a Rickroll HTML page with a fake password checker.

## Challenge Analysis

The PowerShell script creates a simple web server that serves an HTML page containing:
- A Rickroll video/meme
- A password input field
- JavaScript validation code

## Solution

### Step 1: Analyze the JavaScript
The HTML page contains obfuscated JavaScript with a password check:

```javascript
var _0x2a = "WmVyb0RheXN7bjN2M3JfZzBubjRfbDN0X3kwdV9kb3dufQ==";
if (btoa(_0x1a) == _0x2b) { 
    // Success logic
}
```

### Step 2: Identify the Bug
The password check has a bug: `_0x2b` is never defined (undefined variable). This means the password check will always fail, regardless of input.

### Step 3: Extract the Flag
However, `_0x2a` contains a base64-encoded string that looks suspicious:

```python
import base64

encoded = 'WmVyb0RheXN7bjN2M3JfZzBubjRfbDN0X3kwdV9kb3dufQ=='
flag = base64.b64decode(encoded).decode()
print(flag)
# ZeroDays{n3v3r_g0nn4_l3t_y0u_down}
```

## Complete Solution

```python
import base64

# Base64 string found in JavaScript variable _0x2a
encoded_flag = 'WmVyb0RheXN7bjN2M3JfZzBubjRfbDN0X3kwdV9kb3dufQ=='

# Decode to get the flag
flag = base64.b64decode(encoded_flag).decode()
print(flag)
# ZeroDays{n3v3r_g0nn4_l3t_y0u_down}
```

## Technical Details

### The Rickroll Connection
The flag `n3v3r_g0nn4_l3t_y0u_down` is a reference to Rick Astley's "Never Gonna Give You Up" - the famous Rickroll song. This is thematically appropriate given the challenge name `r1ckr0ll.ps1`.

### JavaScript Obfuscation
The JavaScript uses variable name obfuscation (`_0x2a`, `_0x1a`, `_0x2b`) to make the code harder to read. This is a common technique in web-based challenges.

### The Bug
The password validation logic is intentionally broken:
- `_0x1a` is the user input
- `_0x2b` is undefined (never declared)
- The comparison `btoa(_0x1a) == _0x2b` will always be `false`

This suggests the password check is a red herring, and the real flag is hidden in the base64 variable.

## Alternative Approaches

### 1. Browser DevTools
Open the HTML in a browser and use Developer Tools:
- Open Console
- Type `_0x2a` to see the base64 string
- Decode manually or use `atob(_0x2a)` in console

### 2. Source Code Review
Simply read the PowerShell script and extract the HTML, then search for base64 patterns.

### 3. Network Analysis
Run the PowerShell script, access the server, and inspect the page source.

## Lessons Learned

1. **Red herrings are common**: The password check was intentionally broken to mislead
2. **Look for encoded strings**: Base64 strings in JavaScript are often flags
3. **Obfuscation ≠ security**: Variable name obfuscation doesn't hide the data
4. **Thematic clues**: The Rickroll theme hints at the flag content
5. **Check for bugs**: Sometimes the "bug" is intentional and points to the solution

## Tools Used
- **Python**: For base64 decoding
- **Text editor**: To read the PowerShell script
- **Browser DevTools**: Optional for interactive analysis

## References
- [Rickrolling](https://en.wikipedia.org/wiki/Rickrolling)
- [Base64 Encoding](https://en.wikipedia.org/wiki/Base64)
- [JavaScript Obfuscation](https://en.wikipedia.org/wiki/Obfuscation_(software))

---

**Flag**: `ZeroDays{n3v3r_g0nn4_l3t_y0u_down}`

**Solved**: ✅

**Key Takeaway**: In CTF challenges, broken or buggy code is often intentional. When the obvious path doesn't work, look for hidden data in variables, especially base64-encoded strings.
