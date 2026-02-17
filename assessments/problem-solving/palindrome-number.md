# Palindrome Number

## Definition
Determine if a given integer is a palindrome (reads the same forwards and backwards).

## Why it matters
Common interview question to test algorithmic thinking, edge case handling, and basic coding skills.

## When to use / not to use
Use for integer palindrome checks. Not for string or floating-point palindromes.

## Trade-offs
- String conversion is simple but less efficient.
- Arithmetic approach is more efficient but trickier to implement.

## Step-by-step solution
### 1. Clarifying questions
- Are negative numbers considered palindromes? (Usually no)
- Should single-digit numbers return true? (Usually yes)
- Can input be zero? (Yes, and it is a palindrome)

### 2. Approaches
- Convert to string and compare with reverse (easy, O(n) time, O(n) space)
- Reverse half the number arithmetically (O(log10(n)) time, O(1) space)

### 3. Pseudocode
```
if n < 0 or (n % 10 == 0 and n != 0):
    return False
reversed = 0
while n > reversed:
    reversed = reversed * 10 + n % 10
    n //= 10
return n == reversed or n == reversed // 10
```

### 4. Implementations
#### Python
```python
def is_palindrome(n: int) -> bool:
    if n < 0 or (n % 10 == 0 and n != 0):
        return False
    reversed_num = 0
    while n > reversed_num:
        reversed_num = reversed_num * 10 + n % 10
        n //= 10
    return n == reversed_num or n == reversed_num // 10
```
#### Go
```go
func isPalindrome(n int) bool {
    if n < 0 || (n%10 == 0 && n != 0) {
        return false
    }
    reversed := 0
    for n > reversed {
        reversed = reversed*10 + n%10
        n /= 10
    }
    return n == reversed || n == reversed/10
}
```
#### TypeScript
```typescript
function isPalindrome(n: number): boolean {
    if (n < 0 || (n % 10 === 0 && n !== 0)) return false;
    let reversed = 0;
    while (n > reversed) {
        reversed = reversed * 10 + n % 10;
        n = Math.floor(n / 10);
    }
    return n === reversed || n === Math.floor(reversed / 10);
}
```

### 5. Complexity
- Time: O(log₁₀(n))
- Space: O(1)

### 6. Test cases
- 121 → True
- -121 → False
- 10 → False
- 0 → True
- 12321 → True

### 7. Optimizations/Variants
- Use string conversion for simplicity
- Handle very large numbers
- Generalize to string palindromes

### 8. Interview follow-ups
- How would you handle very large numbers?
- Can you do it without converting to string?
- What if the input is a string?

## Prerequisites
- [Basic math](../_index.md)

## Related topics
- [String manipulation](../_index.md)

## Trap questions
1. What happens with negative numbers?
   - They are not palindromes.
2. What about numbers ending in zero?
   - Only zero itself is a palindrome.
3. Can you do it in O(1) space?
   - Yes, with the arithmetic approach.
