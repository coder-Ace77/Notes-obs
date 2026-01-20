
---

There are a few ways to test React components. 
Broadly, they divide into two categories:

- **Rendering component trees** in a simplified test environment and asserting on their output.
- **Running a complete app** in a realistic browser environment (also known as “end-to-end” tests).

Some tools offer a very quick feedback loop between making a change and seeing the result, but don’t model the browser behavior precisely. Other tools might use a real browser environment, but reduce the iteration speed and are flakier on a continuous integration server.

**Jest** is a JavaScript testing framework developed by Facebook. It provides a complete testing solution with features like test runners, assertion libraries, mocking capabilities, and code coverage analysis.

Jest is not specific to React—it's a general-purpose framework that can be used with any JavaScript or TypeScript codebase.

When using Jest, you can write unit tests for individual functions, API calls, and even full React components, but the way components are rendered and interacted with is up to you.

On the other hand, **React Testing Library (RTL)** is a library built specifically for testing React components. It encourages writing tests that resemble how users interact with your app in the browser, rather than testing the internal implementation details. React Testing Library builds on top of Jest (or another test runner), and provides utility functions to render components, query elements, and simulate user interactions such as clicks or typing.

Basics of jest working:

When you run `jest` in your terminal, the first thing it does is **scan your project directory** to find test files. It looks for files that match specific patterns, such as:

- Files with `.test.js`, `.spec.js`, `.test.tsx`, etc.
- Files in a `__tests__` folder
Jest uses configuration options (in `jest.config.js` or `package.json`) to determine which files to consider as test files. It collects all the matching test files and prepares them for execution.

Before executing any tests, Jest sets up a **testing environment**. This is essentially a sandboxed version of a JavaScript runtime that mimics the browser (using **jsdom**) or a Node environment, depending on the test configuration.

Jest supports both:

- **jsdom environment** – simulates a browser, useful for React/component tests
- **Node environment** – used for backend or server-side logic

Note:: Jest doesn’t execute your raw source code directly. 
If you're using modern JavaScript features or TypeScript, Jest integrates with **Babel** or **ts-jest** to transpile your code on the fly.
This means your ES6+, JSX, or TypeScript code is converted into plain JavaScript that the test runner can understand. Jest uses a module called **`babel-jest`** for this by default.

## Testing with jest:

Jest is a test runner and assertion library. However to test react components we need to get the DOM elements this is where RTL(React testing library) comes in.

A sample test file:

```javascript
// 1. Import dependencies
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import MyComponent from './MyComponent';

// 2. Optional: mock any modules or functions
jest.mock('./api'); // if needed

// 3. Group tests using describe()
describe('MyComponent', () => {

  // 4. Write individual test cases
  test('renders heading', () => {
    render(<MyComponent />);
    expect(screen.getByText(/hello/i)).toBeInTheDocument();
  });

  test('calls function on button click', () => {
    const mockFn = jest.fn();
    render(<MyComponent onClick={mockFn} />);
    fireEvent.click(screen.getByText(/click me/i));
    expect(mockFn).toHaveBeenCalled();
  });
});
```

render,screen etc are part of RTL.

1. Render: Render() function is used to mount a react component into virtual DOM(created by JSDOM). Think of it as placing your component on a blank HTML page inside a browser — but in a test environment.
example:

```javascript
import { render } from '@testing-library/react';
import Greeting from './Greeting';

render(<Greeting name="Alice" />);
```

After rendering, the component is mounted, and its DOM is now part of the test environment. Now you can interact with it or check its contents.

The return value of a render is 

```javascript
const { container, getByText, debug } = render(<Greeting name="Alice" />);
```

`container` – the root DOM node of the rendered component
`getByText`, `queryByRole`, etc. – query helpers (deprecated in favor of `screen`)
`debug()` – prints current DOM (great for debugging)

However, instead of destructuring queries, **RTL recommends using `screen`**, as it's globally available and aligns better with real-user behavior.

### screen

The `screen` object provides access to **query functions** like `getByText`, `getByRole`, `findBy`, etc., which help you locate elements in the rendered UI.

- Verify that elements appear (or don’t)
- Simulate user interactions
- Assert text, state, or behavior

```javascript
import { render, screen } from '@testing-library/react';
import Greeting from './Greeting';

render(<Greeting name="Alice" />);
expect(screen.getByText(/hello, alice/i)).toBeInTheDocument();
```

`screen` mirrors how a **user would find elements** — by label, role, placeholder, visible text — not by ID or class (which is discouraged).

|Query|Use Case|
|---|---|
|`getByText()`|Get element by visible text (throws if missing)|
|`queryByText()`|Same as above, but doesn’t throw|
|`getByRole()`|Get element by role (e.g., button, heading)|
|`getByLabelText()`|Get form input by its associated label|
|`findBy...()`|Async version — waits for element to appear|
|`getAllBy...()`|Returns an array (for multiple matches)|
Once the screen can query the element from virtual dom we can use it in jest and then jest testing continues.

```javascript
import { render, screen } from '@testing-library/react';
import Greeting from './Greeting';

test('renders greeting message', () => {
  render(<Greeting name="John" />);
  expect(screen.getByText(/hello, john/i)).toBeInTheDocument();
});
```

## FireEvent:

Its a utility from RTL that lets us dispatch synthetic DOM events(click , change , keydown)etc. Like render its part of react testing library.
Unlike calling `onClick` manually, `fireEvent` goes through the proper bubbling, propagation, and synthetic event handling that React expects.

```javascript
import { fireEvent } from '@testing-library/react';
```

|Method|Description|Example|
|---|---|---|
|`click()`|Simulates a mouse click|`fireEvent.click(button)`|
|`change()`|Simulates changing input value|`fireEvent.change(input, {...})`|
|`input()`|Triggers the `input` event|`fireEvent.input(input, {...})`|
|`keydown()` / `keyup()`|Simulates keyboard key presses|`fireEvent.keyDown(input, {...})`|
|`focus()` / `blur()`|Simulates focus entering or leaving an input|`fireEvent.focus(input)`|
|`submit()`|Simulates form submission|`fireEvent.submit(form)`|
|`dblClick()`|Simulates a double click|`fireEvent.dblClick(button)`|
|`mouseOver()`|Simulates hover|`fireEvent.mouseOver(element)`|


```javascript
test('increments counter on click', () => { render(<Counter />); const button = screen.getByText('Increment'); fireEvent.click(button); expect(screen.getByTestId('count')).toHaveTextContent('1'); });
```

Clicks are simple however to simulate user input we can use change.

```javascript
<input type="text" onChange={(e) => setName(e.target.value)} />

// now test 
fireEvent.change(screen.getByRole('textbox'), { target: { value: 'John' } });
```

Form submit 

```javascript
fireEvent.submit(screen.getByRole('form'));
```

Check boxes:

```javascript
fireEvent.click(screen.getByLabelText('Accept Terms'));
// or
fireEvent.change(screen.getByLabelText('Accept Terms'), { target: { checked: true }});
```


> [!NOTE] Refresher
> Remember that event refers to the whole event object and event.target refers to the input element that triggered the change finally each node in form has value component which has the current value displayed in dom. 


Keydown events:

```javascript
fireEvent.keyDown(input, { key: 'Enter', code: 'Enter' });
```


