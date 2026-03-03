
---

"Unit testing is the practice of testing individual units or components of a program to ensure they work as expected."

#### Unit testing architecture:

1. Unit tests discovery.
2. Test loading
3. Test setup
4. Test execution
5. Assertion evaluation.
6. Error and failure handling
7. Test teardown
8. Result aggregation
9. Reporting


> [!NOTE] Junit recap
> Junit actually is platform for running the tests. It won't run the test by itself. We have another library Jupiter testing which is used to run the tests.  The Jupiter Test Engine specifically handles all the tests annotated with Jupiter’s annotations like `@Test`

### Code coverage:

Code coverage is a way to measure how much of your source code is executed while running tests, giving insights into which parts of your code are tested and which are not.

Internally, code coverage tools work by modifying your program’s bytecode—the compiled form of your Java classes—before or during test execution. This process is called bytecode instrumentation. During instrumentation, the tool injects extra instructions (called probes) into the bytecode at strategic points, such as before each line, branch, or method.

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
```

```java
@Test
public void testAdd() {
    Calculator calc = new Calculator();
    calc.add(2, 3);  // This line executes the add method
}
```

### Testing in javascript using jest:

Jest is a javascript library to test javascript code. 

```js
function sum(a, b) {
  return a + b;
}
```

```js
test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

```js
// calculator.test.js
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

How does expect looks like:

#### Expect at high level

```js

function expect(actualValue) {
  return {
    toBe(expected) {
      if (actualValue !== expected) {
        throw new Error(`Expected ${actualValue} to be ${expected}`);
      }
    },
    toEqual(expected) {
      // Deep equality check here
    },
    toBeGreaterThan(number) {
      // ...
    },
    // ... dozens of other matchers
  };
}
```

The `expect()` function returns a **"matcher object"**, which contains all available **matcher methods** (`toBe`, `toEqual`, etc.)

Matcher : Performs checks and throws error.

#### Now about matcher:

Some default matchers:

1. toBe: It is equivalent to using the `===` operator.

```js
expect(5).toBe(5); // Passes
expect('hello').toBe('hello'); // Passes
expect([1, 2]).toBe([1, 2]); // Fails, different references
```

2. toEqual: Unlike `toBe()`, the `toEqual()` matcher is used for comparing the **values of objects or arrays** deeply

```js
expect({name: 'Alice'}).toEqual({name: 'Alice'}); // Passes
expect([1, 2, 3]).toEqual([1, 2, 3]); // Passes
```

3. toBeNull: unfined will fail.

```js
expect(1).toBeTruthy(); // Passes
expect(0).toBeFalsy(); // Passes
expect('').toBeFalsy(); // Passes
```

4. toContain:

```js
expect('Hello world').toContain('world'); // Passes
expect([1, 2, 3]).toContain(2); // Passes
```

5. - `toBeGreaterThan(x)` , `toBeLessThan(x)` ,  `toBeGreaterThanOrEqual(x)` `toBeLessThanOrEqual(x)`

6. toBeCloseTo():

```js
expect(0.1 + 0.2).toBeCloseTo(0.3); // Passes
```

7. toThrow:

```js
function throwError() {
  throw new Error('Something went wrong');
}

expect(throwError).toThrow(); // Passes
expect(throwError).toThrow('Something went wrong'); // Passes
```

```js
const obj = { user: { name: 'Alice' } };
expect(obj).toHaveProperty('user.name'); // Passes
expect(obj).toHaveProperty('user.name', 'Alice'); // Passes
```

### Chaining modifiers:

```js
expect(5).not.toBe(3); // Passes if 5 !== 3
expect([1, 2, 3]).not.toContain(4); // Passes if 4 is not in the array
```

### Promises:

```js
function getUser() {
  return Promise.resolve({ name: 'Alice' });
}

test('resolves to user object', async () => {
  await expect(getUser()).resolves.toEqual({ name: 'Alice' });
});
```

Reject throws an error:

```js
function getUser() {
  return Promise.reject(new Error('User not found'));
}

test('rejects with error', async () => {
  await expect(getUser()).rejects.toThrow('User not found');
});
```

### Creating custom matcher: 

Use expect.extend api.

```js
expect.extend({
  toBeEven(received) {
    const parity = received % 2 === 0;
    if (parity) {
      return {
        message: () => `expected ${received} not to be even`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected ${received} to be even`,
        pass: false,
      };
    }
  },
});
```

```js
test('checks even numbers', () => {
  expect(4).toBeEven();       
  expect(7).not.toBeEven();
});
```

```js
toBeDivisibleBy(received, divisor) {
  const pass = received % divisor === 0;
  return {
    pass,
    message: () =>
      `expected ${received} ${pass ? 'not ' : ''}to be divisible by ${divisor}`,
  };
}
```

```js
expect(10).toBeDivisibleBy(5);
expect(7).not.toBeDivisibleBy(3);
```

### Testing with data:

```js
test.each([
		[1, 2, 3],
		[2, 3, 5],
		[3, 5, 8]])
		('adds %i + %i to equal %i', (a, b, expected) => {
  expect(a + b).toBe(expected);
});
```

Not so recommended style

```js
const cases = [
  [1, 2, 3],
  [2, 3, 5],
  [3, 5, 8],
];

for (const [a, b, expected] of cases) {
  test(`adds ${a} + ${b} to equal ${expected}`, () => {
    expect(a + b).toBe(expected);
  });
}
```

### Mocking:

**Mocking in Jest** refers to replacing real functions, modules, or components with fake ones in your tests.

Types of mocking:

1. Function mocking with jest.fn():

```js
const mockFn = jest.fn();
mockFn('hello');
expect(mockFn).toHaveBeenCalledWith('hello');
```

This mock function does nothing by default, but it **records every call** made to it, including arguments, return values, and how many times it was called. You can use it in tests to verify interactions, even if it doesn't have a real implementation.

```js
test('calls mock function with correct arguments', () => {
  const mockFn = jest.fn();
  mockFn('hello', 123);
  expect(mockFn).toHaveBeenCalled(); 
  expect(mockFn).toHaveBeenCalledWith('hello', 123); 
  expect(mockFn).toHaveBeenCalledTimes(1); 
});

```

##### Custom return values:

Two ways to do that

```js
const getUser = jest.fn().mockReturnValue({ id: 1, name: 'Alice' });

test('returns a user', () => {
  expect(getUser()).toEqual({ id: 1, name: 'Alice' });
});
```

```js
const add = jest.fn().mockImplementation((a, b) => a + b);

test('adds two numbers', () => {
  expect(add(2, 3)).toBe(5);
});
```

#### Mocking functions from another module:

```js
// math.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

```

```js
// calculator.js
import { add } from './math';

export function calculateTotal(a, b) {
  return add(a, b);
}
```

```js
// calculator.test.js
import { calculateTotal } from './calculator';
import * as math from './math'; 

jest.mock('./math');

test('calculateTotal returns mocked value', () => {
  math.add.mockReturnValue(100);

  const result = calculateTotal(5, 7);

  expect(result).toBe(100); 
  expect(math.add).toHaveBeenCalledWith(5, 7); 
});
```

Every mock function has internal mock field which stores all the history.

```js
const mock = jest.fn((x) => x * 2);

mock(2);
mock(5);

console.log(mock.mock.calls); // [[2], [5]]
console.log(mock.mock.results); // [{type: 'return', value: 4}, {type: 'return', value: 10}]
```

mock object:
```js
{
  calls: [ [arg1, arg2], [arg3, arg4] ],
  results: [ { type: 'return', value: 10 }, { type: 'throw', error: Error } ],
  invocationCallOrder: [1, 2], 
}
```

```js
afterEach(() => {
  jest.clearAllMocks();
}); 
```

Jest can run with two testing environments:

1. Node js environment:  Only backend no document or window api.
2. jsdom environment: (simulated browser) 

### React testing:

**React Testing Library** (RTL) is a **testing utility** for React components.

### Important apis:

`render` is a method provided by `@testing-library/react` that renders a React component into a testing environment (using `jsdom`) so that it can be queried and interacted with in tests.

```js
import { render } from '@testing-library/react';
import MyComponent from './MyComponent';

test('renders my component', () => {
  render(<MyComponent />);
  // Now you can query and interact with the rendered output
});
```

render return signature:

```js
const {
  getByText,
  getByRole,
  getByTestId,
  queryByText,
  findByText,
  container,
  rerender,
  unmount,
  debug,
  ...others
} = render(<MyComponent />);
```

| Function / Property | Description                                                                |
| ------------------- | -------------------------------------------------------------------------- |
| `getByText()`       | Returns the first element with matching text (throws error if not found).  |
| `queryByText()`     | Same as `getByText`, but returns `null` instead of throwing error.         |
| `findByText()`      | Async version of `getByText`, useful for `await`.                          |
| `getByRole()`       | Selects elements by their ARIA role (e.g., `button`, `textbox`).           |
| `container`         | A reference to the rendered DOM container (div).                           |
| `rerender()`        | Allows you to rerender the component with new props.                       |
| `unmount()`         | Unmounts the component from the virtual DOM.                               |
| `debug()`           | Logs the current DOM structure to the console (very useful for debugging). |


### Selecting element from parent:

```js
render(<Card />);
const cardRegion = screen.getByRole('region', { name: /user card/i });

const nameText = within(cardRegion).getByText(/alice/i);
expect(nameText).toBeInTheDocument();
```

using test id

```js
<div data-testid="deep-element">Hello</div>
```

```js
const el = screen.getByTestId('deep-element');
```

### Await:

```js
function WelcomeMessage() {
  const [name, setName] = React.useState('');

  React.useEffect(() => {
    setTimeout(() => setName('Alice'), 500);
  }, []);

  return <div>{name ? `Hello, ${name}!` : 'Loading...'}</div>;
}
```

```js
render(<WelcomeMessage />);

// Wait for the text to appear
const message = await screen.findByText(/hello, alice!/i);

expect(message).toBeInTheDocument();
```

### Wait for:

```js
function Counter() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    const timer = setTimeout(() => setCount(5), 500);
    return () => clearTimeout(timer);
  }, []);

  return <div>Count: {count}</div>;
}
```

```js
render(<Counter />);

await waitFor(() => {
  expect(screen.getByText(/count: 5/i)).toBeInTheDocument();
});
```


### Matchers in RTL:

| Matcher                         | Purpose                                                    | Example                                         |
| ------------------------------- | ---------------------------------------------------------- | ----------------------------------------------- |
| `toBeInTheDocument()`           | Checks if the element exists in the DOM                    | `expect(element).toBeInTheDocument()`           |
| `toHaveTextContent(text)`       | Checks if element contains specific text                   | `expect(element).toHaveTextContent('Hello')`    |
| `toHaveAttribute(attr, value?)` | Checks for presence (and optionally value) of an attribute | `expect(link).toHaveAttribute('href', '/home')` |
| `toBeVisible()`                 | Checks if element is visible to the user                   | `expect(modal).toBeVisible()`                   |
| `toBeDisabled()`                | Checks if a form element is disabled                       | `expect(button).toBeDisabled()`                 |
| `toBeChecked()`                 | Checks if a checkbox/radio is checked                      | `expect(checkbox).toBeChecked()`                |
| `toHaveValue(value)`            | Checks if input has the expected value                     | `expect(input).toHaveValue('Alice')`            |
| `toHaveFocus()`                 | Checks if element has focus                                | `expect(input).toHaveFocus()`                   |
| `toBeEmptyDOMElement()`         | Checks if an element has no children                       | `expect(div).toBeEmptyDOMElement()`             |

### Example:

```js
import React, { useEffect, useState } from 'react';

export default function UserProfile() {
  const [user, setUser] = useState(null);
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/user')
      .then(res => {
        if (!res.ok) throw new Error('Network error');
        return res.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError('Failed to load user');
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p role="alert">{error}</p>;
  return <h1>{user.name}</h1>;
}
```

```js
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import UserProfile from './UserProfile';

beforeEach(() => {
  global.fetch = jest.fn();
});

afterEach(() => {
  jest.resetAllMocks();
});

test('shows user data after fetch', async () => {
  global.fetch.mockResolvedValueOnce({
    ok: true,
    json: async () => ({ name: 'Alice' }),
  });

  render(<UserProfile />);

  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  const heading = await screen.findByRole('heading', { name: /alice/i });
  expect(heading).toBeInTheDocument();

  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();
});
```

Axios exmple

```js
import React, { useEffect, useState } from 'react';
import axios from 'axios';

export default function UserProfile() {
  const [user, setUser] = useState(null);
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    axios.get('/api/user')
      .then(response => {
        setUser(response.data);
        setLoading(false);
      })
      .catch(() => {
        setError('Failed to load user');
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p role="alert">{error}</p>;
  return <h1>{user.name}</h1>;
}

```

Mocking:

```js
import { render, screen } from '@testing-library/react';
import UserProfile from './UserProfile';
import axios from 'axios';

jest.mock('axios'); 
```

```js
test('mock axios testing', async () => {
  axios.get.mockResolvedValueOnce({
    data: { name: 'Alice' },
  });

  render(<UserProfile />);
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  const heading = await screen.findByRole('heading', { name: /alice/i });
  expect(heading).toBeInTheDocument();

  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();
});
```

```js
axios.get.mockResolvedValue({ data: { name: 'Bob' } });
```

```js
axios.get.mockResolvedValueOnce({ data: { name: 'Alice' } })
  .mockResolvedValueOnce({ data: { name: 'Bob' } });
```

```js
axios.get.mockRejectedValue(new Error('Server down'));
```

```js
axios.get.mockImplementation(url => {
  if (url === '/api/user') return Promise.resolve({ data: { name: 'Alice' } });
  else return Promise.reject(new Error('Not found'));
});
```

```js
import React, { useState } from 'react';
import axios from 'axios';

export default function AddItem() {
  const [itemName, setItemName] = useState('');
  const [status, setStatus] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      await axios.post('/api/items', { name: itemName });
      setStatus('success');
    } catch {
      setStatus('error');
    }
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          placeholder="Enter item"
          value={itemName}
          onChange={(e) => setItemName(e.target.value)}
        />
        <button type="submit">Add</button>
      </form>

      {status === 'success' && <p role="alert">Item added!</p>}
      {status === 'error' && <p role="alert">Failed to add item</p>}
    </div>
  );
}

```

```js
test('submits form and shows success message', async () => {
  axios.post.mockResolvedValueOnce({ status: 201 });

  render(<AddItem />);

  await userEvent.type(screen.getByPlaceholderText(/enter item/i), 'Book');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));

  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent(/item added/i);

  expect(axios.post).toHaveBeenCalledWith('/api/items', { name: 'Book' });
});

```

```js
test('shows error message when post fails', async () => {
  axios.post.mockRejectedValueOnce(new Error('Server error'));
  render(<AddItem />);

  await userEvent.type(screen.getByPlaceholderText(/enter item/i), 'Pen');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));

  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent(/failed to add item/i);
});
```

### Events:

`fireEvent` is a **manual way** to dispatch DOM events in tests.

```js
fireEvent.click(element);
fireEvent.change(inputElement, { target: { value: 'New Value' } });
fireEvent.submit(formElement);
fireEvent.focus(inputElement);
```

```js
fireEvent.change(input, { target: { value: 'Alice' } });
expect(input.value).toBe('Alice');
```
