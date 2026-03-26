# Integer Overflow Challenge

## Challenge Information
- **Category**: Binary Exploitation
- **Difficulty**: Medium
- **Points**: 
- **Flag**: `ZeroDays{test_integer_overflow_vulnerability_n3g4t1v3_qty}`

## Description
A binary exploitation challenge involving an integer overflow vulnerability in quantity validation logic.

## Vulnerability Analysis

The challenge binary contained a critical integer overflow vulnerability in its quantity validation mechanism. The program checked if a quantity was negative but failed to account for integer overflow when large positive values were provided.

### Vulnerable Code Pattern
```c
int32_t qty = user_input;
if (qty < 0) {
    return error;  // Reject negative quantities
}
// Continue processing with qty...
```

### The Problem
When a very large positive number (e.g., `2147483648` or `0x80000000`) is provided:
- It overflows when stored as a signed 32-bit integer
- The value wraps around to `-2147483648`
- The negative check passes, but the value is actually negative
- This bypasses the intended security validation

## Solution

### Step 1: Identify the Vulnerability
- Analyzed the binary to find quantity validation logic
- Discovered the program only checked for `qty < 0`
- Realized large positive values would overflow to negative

### Step 2: Trigger the Overflow
- Provided input value: `2147483648` (2^31)
- This value as signed int32: `-2147483648`
- The overflow bypassed the validation check

### Step 3: Exploit the Vulnerability
- Once validation was bypassed, accessed restricted functionality
- Retrieved the flag from memory or file system

## Exploitation

```python
# Pseudocode for exploitation
large_value = 2**31  # 2147483648
# When stored as int32_t, becomes -2147483648
# Passes the qty < 0 check
# Allows access to restricted functionality
```

## Technical Details

### Integer Overflow Mechanics
- **Signed 32-bit integer range**: `-2,147,483,648` to `2,147,483,647`
- **Overflow behavior**: Values beyond max wrap to negative
- **Example**: `2,147,483,648` → `-2,147,483,648`

### Why This Works
1. Input validation checks `if (qty < 0)`
2. Large positive input overflows to negative
3. Negative value passes the check
4. Program continues with corrupted value
5. Restricted functionality becomes accessible

## Prevention

### Proper Input Validation
```c
// Check for overflow before using the value
if (qty < 0 || qty > INT32_MAX) {
    return error;
}

// Or use unsigned integers when negatives aren't valid
uint32_t qty = user_input;
if (qty > MAX_ALLOWED_QTY) {
    return error;
}
```

### Best Practices
1. **Use appropriate data types**: Unsigned integers for quantities
2. **Validate ranges**: Check both minimum and maximum values
3. **Check for overflow**: Before arithmetic operations
4. **Use safe functions**: Compiler-provided overflow detection
5. **Bounds checking**: Always validate input ranges

## Lessons Learned

1. **Integer overflow is subtle**: Easy to miss in code review
2. **Type matters**: Signed vs unsigned makes a difference
3. **Validate thoroughly**: Check all edge cases
4. **Test with extreme values**: Always test boundary conditions
5. **Defense in depth**: Multiple validation layers

## Tools Used
- Binary analysis tools (IDA, Ghidra, or similar)
- Debugger (GDB)
- Python for payload generation

## References
- CWE-190: Integer Overflow or Wraparound
- OWASP: Integer Overflow

---

**Flag**: `ZeroDays{test_integer_overflow_vulnerability_n3g4t1v3_qty}`

