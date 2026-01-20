
---

### Raw vs original string:

The matching string should be a raw string meaning **Backslashes (`\`) are treated as literal characters**, not as escape sequences. Normally `\d` will be a invalid escape sequence but with raw string it can be done pretty easily.

A **Regular Expression (regex)** is a _pattern string_ used to match, find, or replace parts of text.

commonly used re function

| Function        | Purpose                                                    |
| --------------- | ---------------------------------------------------------- |
| `re.match()`    | Check for a match **only at the beginning** of a string    |
| `re.search()`   | Search for the **first occurrence** anywhere in the string |
| `re.findall()`  | Return **all matches** as a list                           |
| `re.finditer()` | Return **iterator of match objects**                       |
| `re.sub()`      | Replace matched patterns                                   |
| `re.split()`    | Split string by pattern                                    |

Any regular expression will take in some args one of them is the regex pattern starting from `r"Pattern string here"`

```python
txt = "Hello World"
m = re.match(r"Hello", txt)
```

Now whatever is returned may depend upon the type of function call.

Any regex string has many meta characters each meta character has its oen meaning. 

`. ^ $ * + ? { } [ ] \ | ( )` to match them litterally one should escape them with backslash `\`

Then there are the charater classes- each class define a set of characters for us

One rule is small letters represent the positive match whereas capital ones represnet the negative match.

|Pattern|Meaning|Example|
|:--|:--|:--|
|`\d`|Any digit `[0-9]`|`\d{3}` → matches 123|
|`\D`|Non-digit|`\D+` → matches `abc`|
|`\w`|Word char `[a-zA-Z0-9_]`|`\w+` → `user123`|
|`\W`|Non-word char|spaces, punctuation|
|`\s`|Whitespace (space, tab, newline)|`\s+`|
|`\S`|Non-whitespace|opposite of `\s`|
Anchors and boundries

| Anchor | Meaning         | Example                                      |
| :----- | :-------------- | :------------------------------------------- |
| `^`    | Start of string | `^Hello`                                     |
| `$`    | End of string   | `world$`                                     |
| `\b`   | Word boundary   | `\bcat\b` → matches `cat`, not `concatenate` |
| `\B`   | Non-boundary    | `\Bcat` → matches `concatenate`              |
and many more
Also if pattern has something which is not character class or quantifiers etc then re will perform exact text matching.

### re.search:

It takes a pattern and a text and returns a match object. If no match happens none is returned otherwise a match object is returned.

match object has span call which returns index location of where match happened. 

```python
m = re.search(pattern,text)

m.span() # tuple (start,end)
m.start() # start of match in text
m.end() # end of amtch in text
m.group() # actual text that matchewd
```

It only gives out only the first match. we have findall() which matches all the occurances

```python 
matches = re.findall(pattern,text) # returns only the match part
```

to get all the match objects individually

```python
for match in re.finditer(pattern,text):
	pass # match here represents single match  
```

#### quantifiers:

They come just after the character identifiers/normal characters and give special meaning for instance + means one or more time of last

| Quantifier | Meaning         | Example  | Matches             |
| ---------- | --------------- | -------- | ------------------- |
| `*`        | 0 or more       | `ab*`    | `a`, `ab`, `abb`    |
| `+`        | 1 or more       | `ab+`    | `ab`, `abb`         |
| `?`        | 0 or 1          | `ab?`    | `a`, `ab`           |
| `{n}`      | exactly n       | `a{2}`   | `aa`                |
| `{n,}`     | n or more       | `a{2,}`  | `aa`, `aaa`         |
| `{n,m}`    | between n and m | `a{2,4}` | `aa`, `aaa`, `aaaa` |
There is also something called as compile it can combine multiple patterns and can look into it one by one.

Matching for one we use pipe operator

```python
re.search(r'cat|dog','the cat is here')
```

`.` opertor matches one character only.

`[]` matches exactly one character in all the characters inside brackets.
`[^]` matches exatly one character not inside
This bracket match one can be extended with quantifiers like `[abc]+` ranges are also allowed
`[a-z0-9]+` etc

Now a part of pattern can be qrouped so that quantifier is applied to that group in entirety.

we can also gp the pattern parts using `()` and then apply quantifiers it will work

`(abcd)+`

```python
re.search(r'[^\d]')
```
### grouping and capturing

