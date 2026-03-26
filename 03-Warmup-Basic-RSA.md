# Warmup/Crypto — Basic RSA with Python

## Challenge Information
- **Category**: Warmup / Cryptography
- **Difficulty**: Easy
- **Points**: Unknown
- **Flag**: `ZeroDays{basic_rsa_with_python}`

## Description
A basic RSA challenge where both prime factors are provided, making decryption straightforward.

## Challenge Data

Given RSA parameters:
```python
p = 214193504765503801665998342866682875591
q = 250786773913245208670191053472027734391
n = 53716898053312012094419829413797156432533681766316762019921021245696645150081
e = 65537
c = 16274262475828245500608792520281893316153511890785081435129136951168480755941
```

## Solution

This is a standard RSA decryption problem. Since both prime factors `p` and `q` are provided, we can easily compute the private key and decrypt the ciphertext.

### Step 1: Calculate Euler's Totient
```python
phi = (p - 1) * (q - 1)
```

### Step 2: Calculate Private Key
```python
from Crypto.Util.number import inverse

d = inverse(e, phi)
```

### Step 3: Decrypt the Ciphertext
```python
m = pow(c, d, n)
```

### Step 4: Convert to Bytes
```python
from Crypto.Util.number import long_to_bytes

flag = long_to_bytes(m)
print(flag)
# b'ZeroDays{basic_rsa_with_python}'
```

## Complete Solution Script

```python
from Crypto.Util.number import inverse, long_to_bytes

# Given parameters
p = 214193504765503801665998342866682875591
q = 250786773913245208670191053472027734391
n = 53716898053312012094419829413797156432533681766316762019921021245696645150081
e = 65537
c = 16274262475828245500608792520281893316153511890785081435129136951168480755941

# Calculate phi (Euler's totient)
phi = (p - 1) * (q - 1)

# Calculate private key d
d = inverse(e, phi)

# Decrypt the ciphertext
m = pow(c, d, n)

# Convert to bytes and print flag
flag = long_to_bytes(m)
print(flag.decode())
# ZeroDays{basic_rsa_with_python}
```

## Technical Details

### RSA Decryption Process
1. **Compute φ(n)**: `φ(n) = (p-1)(q-1)` for RSA with two primes
2. **Compute d**: `d ≡ e⁻¹ (mod φ(n))` using modular inverse
3. **Decrypt**: `m ≡ c^d (mod n)`
4. **Convert**: Convert integer `m` to bytes to get the plaintext

### Why This Works
- RSA security relies on the difficulty of factoring `n` into `p` and `q`
- When both factors are known, the private key can be computed trivially
- This is a warmup challenge demonstrating the basic RSA algorithm

## Lessons Learned

1. **RSA fundamentals**: Understanding the relationship between public/private keys
2. **Modular arithmetic**: Using modular inverse and exponentiation
3. **Python crypto libraries**: Using PyCryptodome for RSA operations
4. **Number theory**: Euler's totient function and its role in RSA

## Tools Used
- **PyCryptodome**: For `inverse()` and `long_to_bytes()` functions
- **Python**: Built-in `pow()` for modular exponentiation

## References
- [RSA Cryptosystem](https://en.wikipedia.org/wiki/RSA_(cryptosystem))
- [Euler's Totient Function](https://en.wikipedia.org/wiki/Euler%27s_totient_function)
- [Modular Multiplicative Inverse](https://en.wikipedia.org/wiki/Modular_multiplicative_inverse)

---

**Flag**: `ZeroDays{basic_rsa_with_python}`

**Solved**: ✅

**Key Takeaway**: RSA decryption is straightforward when both prime factors are known. The security of RSA depends entirely on the difficulty of factoring the modulus.
