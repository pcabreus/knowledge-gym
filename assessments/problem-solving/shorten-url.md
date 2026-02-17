# Shorten URL

## Definition
Design a system or function to shorten a long URL into a short, unique code and retrieve the original URL.

## Why it matters
Classic system design and hashing problem. Tests knowledge of hashing, collisions, and API design.

## When to use / not to use
Use for URL shortening services, unique code generation. Not for cryptographic security.

## Trade-offs
- Simplicity (in-memory map) vs scalability (persistent storage)
- Hash collisions vs sequential IDs

## Step-by-step solution
### 1. Clarifying questions
- Should the same URL always return the same code?
- Is there a limit to the number of URLs?
- How long should the code be?

### 2. Approaches
- Hash the URL and encode (base62)
- Use incremental IDs and encode (base62)
- Store mapping in a dictionary/map

### 3. Pseudocode
```
# Encode
id = next_id()
short = base62(id)
map[short] = url
# Decode
return map[short]
```

### 4. Implementations
#### Python
```python
import string

def encode(num: int) -> str:
    chars = string.digits + string.ascii_letters
    base = len(chars)
    s = ''
    while num > 0:
        s = chars[num % base] + s
        num //= base
    return s or '0'

class URLShortener:
    def __init__(self):
        self.map = {}
        self.counter = 1
    def shorten(self, url: str) -> str:
        code = encode(self.counter)
        self.map[code] = url
        self.counter += 1
        return code
    def restore(self, code: str) -> str:
        return self.map.get(code, '')
```
#### Go
```go
import (
    "strconv"
)

var chars = []rune("0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ")

func encode(num int) string {
    if num == 0 {
        return "0"
    }
    base := len(chars)
    s := ""
    for num > 0 {
        s = string(chars[num%base]) + s
        num /= base
    }
    return s
}

type URLShortener struct {
    Map     map[string]string
    Counter int
}

func NewURLShortener() *URLShortener {
    return &URLShortener{Map: make(map[string]string), Counter: 1}
}

func (u *URLShortener) Shorten(url string) string {
    code := encode(u.Counter)
    u.Map[code] = url
    u.Counter++
    return code
}

func (u *URLShortener) Restore(code string) string {
    return u.Map[code]
}
```
#### TypeScript
```typescript
const chars = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
function encode(num: number): string {
    if (num === 0) return '0';
    let s = '';
    const base = chars.length;
    while (num > 0) {
        s = chars[num % base] + s;
        num = Math.floor(num / base);
    }
    return s;
}

class URLShortener {
    private map: Record<string, string> = {};
    private counter = 1;
    shorten(url: string): string {
        const code = encode(this.counter);
        this.map[code] = url;
        this.counter++;
        return code;
    }
    restore(code: string): string {
        return this.map[code] || '';
    }
}
```

### 5. Complexity
- Time: O(1) for shorten/restore
- Space: O(n) for n URLs

### 6. Test cases
- Shorten and restore "https://example.com" → returns original
- Shorten two different URLs → get two different codes
- Restore unknown code → returns empty string
- Shorten same URL twice → get two codes (with this design)
- Shorten/restore empty string

### 7. Optimizations/Variants
- Use hash of URL for deterministic codes
- Add expiration for codes
- Use persistent storage (DB)
- Add validation for input URLs

### 8. Interview follow-ups
- How to handle collisions?
- How to scale to millions of URLs?
- How to prevent abuse/spam?

## Prerequisites
- [Hashing](../_index.md)
- [Base encoding](../_index.md)

## Related topics
- [API design](../_index.md)
- [Data structures](../_index.md)

## Trap questions
1. What if two URLs hash to the same code?
   - Need collision handling.
2. Can the same URL have multiple codes?
   - Yes, with this design.
3. How to make codes shorter?
   - Use a larger base (base62).
