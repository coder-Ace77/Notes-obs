
---

|Category|Function|Description|Example|
|---|---|---|---|
|**Type Conversion**|`int(x)`|Convert to integer|`int(3.7)` → `3`|
||`float(x)`|Convert to float|`float(5)` → `5.0`|
||`complex(x, y)`|Convert to complex number|`complex(2,3)` → `(2+3j)`|
|**Basic Math**|`abs(x)`|Absolute value|`abs(-7)` → `7`|
||`round(x[, n])`|Round to n decimals|`round(3.1415,2)` → `3.14`|
||`pow(x, y)`|x to the power y|`pow(2,3)` → `8`|
||`divmod(x, y)`|Returns quotient & remainder|`divmod(7,3)` → `(2,1)`|
||`min(x1, x2,...)`|Minimum value|`min(1,5,3)` → `1`|
||`max(x1, x2,...)`|Maximum value|`max(1,5,3)` → `5`|
|**Math Module Functions**|`math.sqrt(x)`|Square root|`math.sqrt(16)` → `4.0`|
||`math.pow(x,y)`|Power (float result)|`math.pow(2,3)` → `8.0`|
||`math.exp(x)`|e^x|`math.exp(1)` → `2.718...`|
||`math.log(x[, base])`|Natural log or log base|`math.log(8,2)` → `3.0`|
||`math.log10(x)`|Base-10 log|`math.log10(100)` → `2.0`|
||`math.sin(x)`|Sine (radians)|`math.sin(math.pi/2)` → `1.0`|
||`math.cos(x)`|Cosine (radians)|`math.cos(0)` → `1.0`|
||`math.tan(x)`|Tangent (radians)|`math.tan(math.pi/4)` → `1.0`|
||`math.degrees(x)`|Radians → degrees|`math.degrees(math.pi)` → `180.0`|
||`math.radians(x)`|Degrees → radians|`math.radians(180)` → `3.1415...`|
||`math.ceil(x)`|Round up to next int|`math.ceil(3.2)` → `4`|
||`math.floor(x)`|Round down|`math.floor(3.8)` → `3`|
||`math.fabs(x)`|Absolute value (float)|`math.fabs(-5.3)` → `5.3`|
||`math.factorial(n)`|Factorial|`math.factorial(5)` → `120`|
||`math.gcd(a,b)`|Greatest common divisor|`math.gcd(12,15)` → `3`|
||`math.isqrt(n)`|Integer square root|`math.isqrt(20)` → `4`|
||`math.trunc(x)`|Truncate to integer|`math.trunc(3.7)` → `3`|
|**Random Numbers**|`random.random()`|Float [0.0,1.0)|`0.3456`|
||`random.randint(a,b)`|Random int [a,b]|`random.randint(1,10)`|
||`random.randrange(a,b[,step])`|Random int with step|`random.randrange(0,10,2)` → even number|
||`random.choice(seq)`|Random element|`random.choice([1,2,3])` → 2|
||`random.shuffle(seq)`|Shuffle in-place|`lst=[1,2,3]; random.shuffle(lst)`|
||`random.uniform(a,b)`|Random float [a,b]|`random.uniform(1.5,3.5)`|
|**Number properties**|`isinstance(x,int/float/complex)`|Check type|`isinstance(3,int)` → `True`|
||`math.isfinite(x)`|True if finite|`math.isfinite(10)` → `True`|
||`math.isinf(x)`|True if infinity|`math.isinf(float('inf'))` → `True`|
||`math.isnan(x)`|True if NaN|`math.isnan(float('nan'))` → `True`|

| Category                      | Function / Method                      | Description                                       | Example                                          |
| ----------------------------- | -------------------------------------- | ------------------------------------------------- | ------------------------------------------------ |
| **Basic operations**          | `len(s)`                               | Length of string                                  | `len("hello")` → `5`                             |
|                               | `str(s)`                               | Convert to string                                 | `str(123)` → `"123"`                             |
|                               | `+`                                    | Concatenate                                       | `"hello" + " world"` → `"hello world"`           |
|                               | `*`                                    | Repeat                                            | `"ha"*3` → `"hahaha"`                            |
|                               | `in` / `not in`                        | Membership test                                   | `'a' in 'cat'` → `True`                          |
| **Case methods**              | `s.upper()`                            | Convert to uppercase                              | `"abc".upper()` → `"ABC"`                        |
|                               | `s.lower()`                            | Convert to lowercase                              | `"ABC".lower()` → `"abc"`                        |
|                               | `s.title()`                            | Capitalize first letter of words                  | `"hello world".title()` → `"Hello World"`        |
|                               | `s.capitalize()`                       | Capitalize first char only                        | `"hello world".capitalize()` → `"Hello world"`   |
|                               | `s.swapcase()`                         | Swap case                                         | `"AbC".swapcase()` → `"aBc"`                     |
| **Search / find**             | `s.find(sub[, start[, end]])`          | Return lowest index of substring (or -1)          | `"hello".find('l')` → `2`                        |
|                               | `s.rfind(sub[, start[, end]])`         | Return highest index                              | `"hello".rfind('l')` → `3`                       |
|                               | `s.index(sub[, start[, end]])`         | Like `find`, but raises `ValueError` if not found | `"hello".index('e')` → `1`                       |
|                               | `s.rindex(sub[, start[, end]])`        | Reverse index, raises error if not found          | `"hello".rindex('l')` → `3`                      |
|                               | `s.count(sub[, start[, end]])`         | Count occurrences                                 | `"hello".count('l')` → `2`                       |
|                               | `s.startswith(prefix[, start[, end]])` | Check prefix                                      | `"hello".startswith('he')` → `True`              |
|                               | `s.endswith(suffix[, start[, end]])`   | Check suffix                                      | `"hello".endswith('lo')` → `True`                |
| **Whitespace / trimming**     | `s.strip([chars])`                     | Remove leading/trailing chars or whitespace       | `" hi ".strip()` → `"hi"`                        |
|                               | `s.lstrip([chars])`                    | Remove leading                                    | `" hi ".lstrip()` → `"hi "`                      |
|                               | `s.rstrip([chars])`                    | Remove trailing                                   | `" hi ".rstrip()` → `" hi"`                      |
| **Splitting / joining**       | `s.split(sep=None, maxsplit=-1)`       | Split string into list                            | `"a,b,c".split(',')` → `['a','b','c']`           |
|                               | `s.rsplit(sep=None, maxsplit=-1)`      | Split from right                                  | `"a,b,c".rsplit(',',1)` → `['a,b','c']`          |
|                               | `s.splitlines([keepends])`             | Split by newline                                  | `"a\nb\nc".splitlines()` → `['a','b','c']`       |
|                               | `sep.join(iterable)`                   | Join elements with separator                      | `','.join(['a','b','c'])` → `"a,b,c"`            |
| **Replacement / translation** | `s.replace(old, new[, count])`         | Replace substrings                                | `"hello".replace('l','x')` → `"hexxo"`           |
|                               | `s.translate(table)`                   | Replace using mapping table                       | `s.translate(str.maketrans('abc','xyz'))`        |
|                               | `str.maketrans(x,y)`                   | Create translation table                          | `'abc'.maketrans('abc','xyz')`                   |
| **Checks / predicates**       | `s.isalpha()`                          | True if all letters                               | `"abc".isalpha()` → `True`                       |
|                               | `s.isdigit()`                          | True if all digits                                | `"123".isdigit()` → `True`                       |
|                               | `s.isalnum()`                          | Letters or digits                                 | `"abc123".isalnum()` → `True`                    |
|                               | `s.islower()`                          | All lowercase                                     | `"abc".islower()` → `True`                       |
|                               | `s.isupper()`                          | All uppercase                                     | `"ABC".isupper()` → `True`                       |
|                               | `s.isspace()`                          | Only whitespace                                   | `" ".isspace()` → `True`                         |
|                               | `s.istitle()`                          | Each word capitalized                             | `"Hello World".istitle()` → `True`               |
| **Formatting**                | `s.format(*args, **kwargs)`            | Insert values                                     | `"Hello, {}".format("Alice")` → `"Hello, Alice"` |
|                               | `f"{var}"`                             | f-string formatting                               | `name="Alice"; f"Hi {name}"` → `"Hi Alice"`      |
|                               | `s.zfill(width)`                       | Pad with zeros left                               | `"42".zfill(5)` → `"00042"`                      |
|                               | `s.center(width[, fillchar])`          | Center string                                     | `"hi".center(6,'-')` → `"--hi--"`                |
|                               | `s.ljust(width[, fillchar])`           | Left-justify                                      | `"hi".ljust(5,'*')` → `"hi***"`                  |
|                               | `s.rjust(width[, fillchar])`           | Right-justify                                     | `"hi".rjust(5,'*')` → `"***hi"`                  |
| **Encoding / decoding**       | `s.encode(encoding='utf-8')`           | Encode string to bytes                            | `"hi".encode()` → `b'hi'`                        |
|                               | `bytes.decode(encoding='utf-8')`       | Decode bytes to string                            | `b'hi'.decode()` → `"hi"`                        |