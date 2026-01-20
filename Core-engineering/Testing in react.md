
---

### React testing:

**React Testing Library** (RTL) is a **testing utility** for React components. It allows you to test components in a way that **mimics how users interact** with your app — by querying the DOM and simulating real events. It's part of the broader **Testing Library** family (e.g., DOM Testing Library, Vue Testing Library, etc.), and it's commonly used with **Jest** as the test runner.

### Core philosphy:

Instead of testing **implementation details**, RTL encourages testing **what the user sees and does**:

- Render the component into a **JSDOM**-based DOM.
- Query the DOM for visible elements (like buttons, headings, inputs).
- Simulate **user interactions** (click, type, etc.).
- Assert that the UI updates as expected.

Important functions:

### `render()`

The `render()` function is one of the most fundamental parts of React Testing Library. It is used to render a React component into a special in-memory “virtual” DOM that simulates how it would behave in a real browser environment. When you call `render()`, it returns a set of utilities—such as `getByText`, `queryByRole`, and more—that allow you to inspect and interact with the component in your tests. This function makes it possible to test components as users would use them, by focusing on the rendered output rather than the internal implementation. The rendered component remains in the DOM (specifically within `document.body`) for the duration of the test, and this DOM is what you interact with using queries and simulated user interactions.

```js
// MyComponent.test.jsx
import { render } from '@testing-library/react';
import MyComponent from './MyComponent';

test('renders the component', () => {
  const { getByText } = render(<MyComponent />);
  expect(getByText('Hello, world!')).toBeInTheDocument();
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

|Function / Property|Description|
|---|---|
|`getByText()`|Returns the first element with matching text (throws error if not found).|
|`queryByText()`|Same as `getByText`, but returns `null` instead of throwing error.|
|`findByText()`|Async version of `getByText`, useful for `await`.|
|`getByRole()`|Selects elements by their ARIA role (e.g., `button`, `textbox`).|
|`container`|A reference to the rendered DOM container (div).|
|`rerender()`|Allows you to rerender the component with new props.|
|`unmount()`|Unmounts the component from the virtual DOM.|
|`debug()`|Logs the current DOM structure to the console (very useful for debugging).|

- **`render()` creates a DOM container**: It uses `document.createElement('div')` and appends it to a virtual DOM using `jsdom`.
- **It renders the React component** inside that container using `ReactDOM.render()` (or internally via `createRoot` if using React 18+).
- **It wraps the container** in RTL’s custom logic to allow querying and interaction.


In the context of **React Testing Library (RTL)**, the `container` is a **DOM element (usually a `<div>`)** where your React component gets rendered during a test. It's part of the object returned by the `render()` function:

```js
const { container } = render(<MyComponent />);
```

While RTL encourages using **queries like `getByRole`, `getByText`, etc.** (to mimic how users find elements), `container` gives you **direct access to the DOM**.

```js
const button = container.querySelector('button');
expect(button).toBeTruthy();
```

### Screen:

`screen` is an object provided by **React Testing Library** that gives you access to all the **query functions** used to find elements in your rendered component — without needing to destructure them from `render()`.

```js
render(<MyComponent />);
screen.getByRole('button');
```

|Function|Description|
|---|---|
|`getByText`|Finds element by visible text|
|`getByRole`|Finds element by ARIA role (e.g., button, textbox)|
|`getByLabelText`|Finds input by associated `<label>`|
|`getByPlaceholderText`|Finds by input placeholder|
|`getByAltText`|Finds image by alt text|
|`getByTitle`|Finds by `title` attribute|
|`getByTestId`|Finds by `data-testid` attribute (only if no better option)|

1. **getBy...** (synchronous, throws if not found)
2. **queryBy...** (synchronous, returns `null` if not found)Same as `getBy...`, but **does not throw** an error if the element is missing.
3. Use when you're waiting for **async changes**, like after a loading spinner or API call.

```js
await screen.findByText(/loaded/i);
```

> Internally uses `waitFor()` to keep checking until the element appears or times out.

4.  . **getAllBy / queryAllBy / findAllBy**

These are the plural versions — they return an **array** of matching elements.

```js
const items = screen.getAllByRole('listitem');
expect(items).toHaveLength(3);
```

### Get by Role:

**Accessible Rich Internet Applications**
`getByRole` searches the rendered DOM for an element with a specific **ARIA role**.
**ARIA roles** define what kind of element something is, from an accessibility standpoint. For example:

- `button` — a clickable button
- `textbox` — an input or textarea field
- `heading` — a heading element (like h1, h2, etc.)
- `link` — a hyperlink

any div can also attain an aria role.

```js
<button>Click me</button> <!-- implicit role="button" -->
<div role="button">Click me</div> <!-- explicit role="button" -->
```

```js
const button = screen.getByRole('button', { name: /submit/i });
```

- The **accessible name** is the string that screen readers use to identify the element.
- This is what you usually provide as the `{ name: /text/i }` option in `getByRole`.
- The **visible text content** (e.g., the button’s label)
-  The value of `aria-label` attribute
- The content of an associated `<label>` (for form inputs)

```js
<button aria-label="Close dialog">X</button>
```

- `aria-checked` — for checkboxes and radio buttons (true/false/mixed)
- `aria-disabled` — marks an element as disabled
- `aria-expanded` — indicates if expandable content is open
- `aria-haspopup` — indicates presence of a popup menu
- `aria-selected` — marks selected elements in a group
- `aria-hidden` — hides elements from assistive tech

# What do you get when you select an element?

- The result of queries like `getByText()`, `getByRole()`, or `findByRole()` is a **DOM node** — an **HTMLElement** (or a subclass like HTMLButtonElement).
- This is basically a reference to the actual HTML element inside the **virtual DOM** React Testing Library uses.
Now we can assert the presence of this element in the DOM.

```js
const button = screen.getByRole('button', { name: /submit/i });
expect(button).toBeInTheDocument(); // element exists
expect(button).toBeVisible();       // element is visible
expect(button).toBeEnabled();       // not disabled
expect(button).toHaveClass('btn-primary'); // has CSS class
```

```js
expect(button.textContent).toBe('Submit');
expect(button.getAttribute('type')).toBe('submit');
expect(button).toHaveAttribute('aria-label', 'Submit form');
```

### find

- `findByText` is an **asynchronous** query function.
- It **returns a Promise** that resolves when an element with the specified **visible text** is found in the DOM.
- If the element doesn’t appear within a timeout, it **rejects (throws an error)**.

```js
const element = await screen.findByText(text, options?, waitForOptions?);
```

Working:

- React Testing Library repeatedly calls `getByText` behind the scenes.
- It keeps checking the DOM at intervals (default ~50ms).
- If it finds a matching element, it resolves the Promise with that element.
- If the timeout (default 1000ms) is exceeded without finding the element, it throws.

```js
function UserGreeting() {
  const [name, setName] = React.useState(null);

  React.useEffect(() => {
    setTimeout(() => {
      setName('Alice');
    }, 500);
  }, []);

  // return <div>{name ? `Hello, ${name}!` : 'Loading...'}</div>;
}
```

```js
import { render, screen } from '@testing-library/react';

test('shows greeting after loading', async () => {
  render(<UserGreeting />);

  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Wait for the greeting text to appear asynchronously
  const greeting = await screen.findByText(/hello, alice!/i);
  expect(greeting).toBeInTheDocument();
});

```


```js
// Case-sensitive, exact match:
await screen.findByText('Submit', { exact: true });

// Custom timeout (e.g., wait up to 3000ms)
await screen.findByText(/loaded/i, {}, { timeout: 3000 });

```

