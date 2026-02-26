
---

Pre read: React testing

Jest can work with any of javascript run time. On backend it can work with node run time and on front end it can use `jsdom` to mock the browser.

### Basic test

Consider a file which we want to test.

```javascript
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

We can test it using a .test.js file

```javascript
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

We can add a script in our package to do tests.

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

Now we can run npm test.

Note that jest requires the plain javascript on ES6 or typesdcript to run so we need to configure if using any of the above.

[ts-jest](https://github.com/kulshekhar/ts-jest) is a TypeScript preprocessor with source map support for Jest that lets you use Jest to test projects written in TypeScript.

### Anatomy of test files:

At top one may be required to import some testing dependencies.

```javascript
const { sum } = require('./math'); // module under test
```

We can describe test suites using describe() blocks. It means we club together the related tests.

```javascript
describe('sum()', () => {
  // tests go here
});
```

Each individual test is defined using the test() or it() which is an alias.

```javascript
test('adds 2 + 2 to equal 4', () => {
  expect(sum(2, 2)).toBe(4);
});  // test takes two params description and a function where matching happens.
```

##### Full example:

```javascript
const { add, divide } = require('./calculator');

describe('Calculator', () => {
  beforeEach(() => {
    // Setup if needed
  });

  test('adds two numbers correctly', () => {
    expect(add(2, 3)).toBe(5);
  });

  test('throws when dividing by zero', () => {
    expect(() => divide(4, 0)).toThrow('Cannot divide by zero');
  });

  describe('nested group: divide()', () => {
    test('divides two numbers correctly', () => {
      expect(divide(10, 2)).toBe(5);
    });
  });
});
```

### Matchers:

Jest uses matchers to let you test values in different ways. Consider it to be the equivalent of `assertCondition`.

1. Testing equality

```javascript
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

expect() actually returns an expectation object on which a matcher is called here toBe() is a matcher.

```javascript
expectationObject.toBe(expectation)
```

2. Testing objects:

```javascript
test('object assignment', () => {
  const data = {one: 1};
  data['two'] = 2;
  expect(data).toEqual({one: 1, two: 2});
});
```

It recursively checks every field of an object or array.

`toEqual` ignores object keys with `undefined` properties, `undefined` array items, array sparseness, or object type mismatch. To take these into account use `toStrictEqual` instead.

Not testing

```javascript
test('adding positive numbers is not zero', () => {
  for (let a = 1; a < 10; a++) {
    for (let b = 1; b < 10; b++) {
      expect(a + b).not.toBe(0);
    }
  }
});
```

We can explicitly also match 

3. null , undefined etc

```javascript
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```

4. Numbers:

Most ways of comparing numbers have matcher equivalents.

```javascript
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe and toEqual are equivalent for numbers
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```

For floating point equality, use `toBeCloseTo` instead of `toEqual`, because you don't want a test to depend on a tiny rounding error.

```javascript
test('adding floating point numbers', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);           This won't work because of rounding error
  expect(value).toBeCloseTo(0.3); // This works.
});
```

5. Strings

Strings can be tested using regular expressions.Jest uses **JavaScript’s native RegExp syntax**, which is based on **ECMAScript Regular Expressions**.

```javascript
test('email contains @ symbol', () => {
  expect('user@example.com').toMatch(/@/);
});
```

We can very easily use regualr expression in toMatch 
rule: /substringToLookFor/

Note that any regular expression must come inside `//` pair


| **Pattern** | **Meaning**                                             | **Example**                            |
| ----------- | ------------------------------------------------------- | -------------------------------------- |
| `.`         | Matches any character except newline                    | `/c.t/` matches `cat`, `cot`, etc.     |
| `^`         | Matches beginning of string                             | `/^Hello/` matches `Hello world`       |
| `$`         | Matches end of string                                   | `/world$/` matches `Hello world`       |
| `*`         | Matches 0 or more of the preceding character            | `/lo*/` matches `l`, `lo`, `loo`       |
| `+`         | Matches 1 or more of the preceding character            | `/lo+/` matches `lo`, `loo`            |
| `?`         | Matches 0 or 1 of the preceding character               | `/lo?/` matches `l`, `lo`              |
| `\`         | Escapes special characters                              | `/\./` matches a literal `.`           |
| `[...]`     | Character class: matches any character in brackets      | `/[aeiou]/` matches a vowel            |
| `[^...]`    | Negated character class                                 | `/[^aeiou]/` matches consonants        |
| `(a         | b)`                                                     | Alternation: matches `a` or `b`        |
| `{n}`       | Matches exactly n times                                 | `/a{3}/` matches `aaa`                 |
| `{n,}`      | Matches n or more times                                 | `/a{2,}/` matches `aa`, `aaa`, etc.    |
| `{n,m}`     | Matches between n and m times                           | `/a{2,4}/` matches `aa`, `aaa`, `aaaa` |
| `\d`        | Matches any digit (0–9)                                 | `/\d/` matches `3`                     |
| `\D`        | Matches any non-digit                                   | `/\D/` matches `a`, `@`, etc.          |
| `\w`        | Matches any word character (alphanumeric or underscore) | `/\w/` matches `a`, `1`, `_`           |
| `\W`        | Matches any non-word character                          | `/\W/` matches `@`, `#`, etc.          |
| `\s`        | Matches any whitespace character                        | `/\s/` matches space, tab, etc.        |
| `\S`        | Matches any non-whitespace character                    | `/\S/` matches `a`, `1`, `!`, etc.     |

> [!NOTE] REgex literal vs string pattern
> A **regex literal** is a way to directly define a regular expression using special syntax, typically between forward slashes (`/pattern/`) in languages like JavaScript.- This creates a regular expression that matches the word `"hello"`.
> - No quotes are used — it's a _literal_ regex object. 
> - Special regex characters (like `\d`, `\w`, `^`, `$`, etc.) work as expected without needing extra escaping.
> A **string pattern** is a regular expression written as a **string**. You often pass it into a function or constructor like `RegExp()` in JavaScript

```javascript
const regex = new RegExp("hello");
```

#### Matching arrays:

You can check if an array or iterable contains a particular item using `toContain`:

```javascript
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'milk',
];

test('the shopping list has milk on it', () => {
  expect(shoppingList).toContain('milk');
  expect(new Set(shoppingList)).toContain('milk');
});
```

Checking for exceptions.

toThrow can match errors.

```javascript
function compileAndroidCode() {
  throw new Error('you are using the wrong JDK!');
}

test('compiling android goes as expected', () => {
  expect(() => compileAndroidCode()).toThrow();
  expect(() => compileAndroidCode()).toThrow(Error);
  
  expect(() => compileAndroidCode()).toThrow('you are using the wrong JDK');
  expect(() => compileAndroidCode()).toThrow(/JDK/);

  expect(() => compileAndroidCode()).toThrow(/^you are using the wrong JDK!$/); // Test pass
});
```

The function that throws an exception needs to be invoked within a wrapping function otherwise the `toThrow` assertion will fail. Internally, when you write `expect(fn).toThrow()`, Jest wraps the provided function in a **try-catch block** to monitor if an error is thrown.

### Promises:

Suppose a function fetchData returns the promise.

So we can test it using then call

```javascript
test('the data is peanut butter', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```

Alternatively we can test async/await:

```javascript
test('the data is peanut butter', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (error) {
    expect(error).toMatch('error');
  }
});
```

We can combine async/await with .resolves/.rejects

```javascript
test('the data is peanut butter', async () => {
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  await expect(fetchData()).rejects.toMatch('error');
});

test('the data is peanut butter', () => {
  return expect(fetchData()).resolves.toBe('peanut butter');
});
```

Be sure to return the assertion—if you omit this `return` statement, your test will complete before the promise returned from `fetchData` is resolved and then() has a chance to execute the callback.

## Setup:

Set up refers to the process of doing some work we may be needed to do for tests. We can use beforeEach and afterEach hook.

```javascript
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

beforeEach and afterEach works similar as async code. For example, if `initializeCityDatabase()` returned a promise that resolved when the database was initialized, we would want to return that promise:

```javascript
beforeEach(() => {
  return initializeCityDatabase();
});
```

Similarly we have before all and after all

```javascript
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

#### Mocking functions:

**Mocking** is a testing technique used to simulate (or "fake") the behavior of real functions, modules, or APIs in a controlled way during unit testing. The idea is to **replace real dependencies** with mock versions so that tests can run **faster, more reliably, and in isolation** — without relying on actual implementations like databases, HTTP calls, or file systems.

There are two ways to mock functions: Either by creating a mock function to use in test code, or writing a [`manual mock`](https://jestjs.io/docs/manual-mocks) to override a module dependency.

You can mock a simple function using `jest.fn()`

```javascript
const greet = jest.fn(); // mock
greet("John");

test("greet is called", () => {
  expect(greet).toHaveBeenCalledWith("John");
});
```

jest.fn() takes a function as parameter call and returns the functionality of call.

```javascript
const add = jest.fn((a, b) => a + b);

test("add function", () => {
  expect(add(2, 3)).toBe(5);
});
```

To mock the returned value we can use following

```javascript
const getUser = jest.fn().mockReturnValue({ id: 1, name: "Alice" }); // mock

test("mock return value", () => {
  expect(getUser()).toEqual({ id: 1, name: "Alice" });
});
```

We can also mock promises:

```javascript
const fetchData = jest.fn().mockResolvedValue({ success: true }); // mocks the promise

test("mock resolved promise", async () => {
  await expect(fetchData()).resolves.toEqual({ success: true });
});
```

### jest.mock()

`jest.mock()` is a powerful utility used to **automatically or manually mock a module or dependency**. 
When you call `jest.mock('module-name')`, Jest intercepts any import of that module and replaces it with a **mock version**. By default, this mock is an **automatically generated mock**, meaning Jest replaces all exported functions or classes with dummy functions (`jest.fn()`), which you can later control using `.mockReturnValue()`, `.mockResolvedValue()`, etc. You can also supply a custom implementation by passing a factory function as the second argument to `jest.mock()`.

example:

Suppose mocking this

```javascript
export const fetchData = () => {
  return fetch('https://api.example.com/data');
};
```

```javascript
import { fetchData } from './api';

jest.mock('./api');

test('mock fetchData', () => {
  fetchData.mockReturnValue('mocked data');
  expect(fetchData()).toBe('mocked data');
});
```

we can also customise the mock:

```javascript
jest.mock('./api', () => ({
  fetchData: jest.fn(() => 'custom mocked data'),
}));
```

A **customized mock implementation** in Jest allows you to **manually define how a mocked module or function should behave**, rather than relying on Jest's default auto-mocking. This is especially useful when your tests depend on **specific return values**, **conditional behavior**, or you only want to mock **part** of a module.

You can pass a **factory function** as the second argument to `jest.mock()`. This function returns an object that mocks the module's exports with your custom implementations.

Example:

```javascript
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
```

Now we will try to mock above module.

As can be observed that we tried to mock math module -> it was not api call.
jest.mock('.module_name',()=>({

}))

second argument is a factory method which can create various methods.

```javascript
jest.mock('./math', () => ({
  add: jest.fn(() => 100),             // always returns 100
  subtract: jest.fn((a, b) => a * b),  // multiplies instead of subtracts
}));

import { add, subtract } from './math';

test('custom mock add', () => {
  expect(add(2, 3)).toBe(100);           // custom behavior
});

test('custom mock subtract', () => {
  expect(subtract(2, 3)).toBe(6);        // not the real implementation
});
```

## Testing in react:

Example component

```javascript
// UserGreeting.js
import React, { useState, useEffect } from 'react';

export function UserGreeting({ fetchUser }) {
  const [name, setName] = useState('');

  useEffect(() => {
    fetchUser().then((user) => {
      setName(user.name);
    });
  }, [fetchUser]);

  return (
    <div>
      {name ? <h1>Hello, {name}!</h1> : <p>Loading...</p>}
    </div>
  );
}
```

Observe that we have mocked the behaviour of fetchUser here.

```javascript
// UserGreeting.test.js
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import { UserGreeting } from './UserGreeting';

test('displays loading and then user name after fetch', async () => {
  const mockFetchUser = jest.fn().mockResolvedValue({ name: 'Alice' });

  render(<UserGreeting fetchUser={mockFetchUser} />);

  // Initial render should show "Loading..."
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Wait for the useEffect to resolve
  await waitFor(() => {
    expect(screen.getByText(/hello, alice/i)).toBeInTheDocument();
  });

  // Make sure the fetch function was called
  expect(mockFetchUser).toHaveBeenCalledTimes(1);
});

```

