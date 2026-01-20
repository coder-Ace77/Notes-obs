
---

YAML (short for "YAML Ain't Markup Language") is a human-readable data serialization format commonly used for configuration files, data exchange between languages, and storing hierarchical data.

YAML uses indentation and minimal syntax to represent complex data structures like lists, dictionaries (key-value pairs), and nested objects. YAML files use indentation (usually 2 spaces) to represent structure and hierarchy. Indentation replaces the brackets or braces seen in JSON or XML.

### Syntax rules:

#### 1. Key value pairs:

Most basic unit where a key is followed by a colon and value.

```yaml
name: John Doe
age: 30
```

Keys are strings and values are strings, numbers, boolean, lists, or other mappings. Values can also be other full fledged objects.

#### 2. Lists:

Lists are represented by dashes `-` followed by a space and the list item:

```yaml
fruits:
  - Apple
  - Banana
  - Cherry
```

#### 3.Nested structures:

```yaml
person:
  name: John
  age: 30
  hobbies:
    - reading
    - hiking
```

Note: comments are defined by # 

#### Rules:

- **Indentation matters**: Indentation defines the structure and must be consistent (usually 2 spaces per level). Tabs are not allowed.
- **No quotation marks needed for strings**: Strings don’t need quotes unless they contain special characters or start with symbols like `@`, `#`, or include colons.
- **Keys must be unique within the same level**: You cannot have duplicate keys in the same mapping.
- **Scalars (basic values)** can be plain strings, numbers, booleans (`true`/`false`), or null (`null`, `~`).
- **Multiline strings** can be created using the pipe `|` (literal block scalar) or greater-than `>` (folded block scalar):

```yaml
description: |
  This is a
  multiline string
  that preserves line breaks.

summary: >
  This is a
  folded string
  where line breaks
  are replaced by spaces.
```

#### Type casting:

As we have seen values are automatically type casted but sometimes we may be required to convert one type to another before that we have to learn about the rules.

YAML supports various data types, and sometimes automatic type detection can cause unexpected behavior. Important data types include:

- **Strings** (default, unless the value looks like a number or boolean)
- **Numbers** (integer and floating-point)
- **Booleans** (`true`, `false`, `yes`, `no`)
- **Null values** (`null`, `~`, empty)
- **Dates** (e.g., `2024-08-21` can be interpreted as a date, which may cause issues if you wanted it as a string)

So if u want to treat for example date as string then one can explicitly wrap the value in quotes.

Common points::

1. - **Using tabs instead of spaces** — YAML strictly forbids tabs; always use spaces.
2. - **Incorrect indentation** — even one space can break your structure.
3. **Forgetting to quote strings that look like booleans or numbers** — e.g., `yes` might be interpreted as `true` unless quoted.
4. **Duplicate keys** — YAML does not allow duplicate keys in the same mapping; this can cause unexpected overrides or errors.