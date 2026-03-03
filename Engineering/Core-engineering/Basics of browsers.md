
---

Main components of browser are

1. HTML parser: Reads HTML text and converts it into a **DOM Tree** (Document Object Model).

2. DOM: A **tree-like structure** representing the contents of a web page.- Every element (like `<div>`, `<p>`, etc.) is a node in this tree.The `document` object in JavaScript gives you access to this tree.

```js
document.getElementById('title');
```

3. **CSSOM (CSS Object Model)**Similar to the DOM, but for CSS. It’s how the browser interprets and applies CSS rules to elements.Combines with the DOM to compute the final styles on each element. 

4. Render Engine / Layout Engine: Takes the DOM and CSSOM and **lays out elements on the screen**. Calculates where every element goes and how it looks. This includes things like color, font size, spacing, etc.

5. JavaScript Engine - Executes JavaScript code. Examples: Chrome uses **V8** Firefox uses **SpiderMonkey** Safari uses **JavaScriptCore**.

6. **Window Object** : The **global object** in the browser. Represents the browser tab or window. All global variables and functions live on `window`.

```js
console.log(window.document === document); // true
```

7. **Event Loop & Web APIs**
- JavaScript in browsers is **single-threaded**, but async operations (like `setTimeout` or `fetch`) are handled by **Web APIs**.
- The **event loop** processes callbacks from these APIs once the main thread is idle.


---

### 1. **Introduction to the Browser**

A web browser is a software application that retrieves, interprets, and displays content from the World Wide Web. It converts HTML, CSS, and JavaScript into an interactive visual interface. Popular browsers include Chrome, Firefox, Safari, and Edge. The core job of a browser is to take user input (like a URL or a click), fetch resources from the internet, render them, and handle user interaction.
### 2. **Browser Architecture Overview**

The browser has several key components:

- **User Interface**: Everything the user interacts with (address bar, back/forward buttons, etc.).
- **Browser Engine**: Bridges the UI and the rendering engine.
- **Rendering Engine**: Responsible for parsing HTML/CSS and painting the content on the screen.
- **Networking Layer**: Handles network calls like HTTP requests.
- **JavaScript Engine**: Executes JavaScript code.
- **Data Storage**: LocalStorage, SessionStorage, Cookies, IndexedDB, Cache API, etc.

### 3. **Loading a Web Page**

When a user enters a URL and presses Enter:

1. The browser parses the URL and performs a **DNS lookup** to resolve the domain to an IP address.
2. It initiates a **TCP connection** (often secured via HTTPS using TLS).
3. The browser sends an **HTTP GET** request to the server.
4. The server responds with an **HTML document**, which the browser begins to parse and render.

### 4. **Parsing and Rendering**

The browser reads the HTML and builds a **DOM (Document Object Model)** tree. Simultaneously, it parses CSS and builds a **CSSOM (CSS Object Model)**. These trees are combined to create a **Render Tree**. The browser then:

- Calculates layout (where elements go on screen).
- Paints elements (converts them into pixels).
- Composites the final output on the screen.
If it encounters `<script>` tags, it pauses parsing and sends the script to the JavaScript engine (like V8 in Chrome) to execute.

### 5. **JavaScript Execution and Event Loop**

The **JavaScript engine** executes scripts in a **single-threaded** environment using the **event loop** model. It handles:

- **Call Stack**: Where functions are executed.
- **Web APIs**: Provided by the browser, not JavaScript itself (e.g., `setTimeout`, `fetch`, DOM events).
- **Callback Queue**: Where async callbacks wait until the call stack is free.
- **Event Loop**: Monitors the stack and queue, pushing tasks to the stack when it’s empty.

This is how browsers handle **asynchronous operations** like timers, fetch requests, and event listeners.

### 6. **Handling Events**

Browsers use an **event-driven model**. Events (like clicks, keypresses, mouse moves) are captured by the browser and pushed into the event queue. JavaScript code can register **event listeners** (e.g., `element.addEventListener`) to handle these events. When the call stack is clear, the event loop picks an event callback from the queue and executes it.

This allows the browser to remain responsive, even when waiting for user interaction or network responses.
### 7. **Fetch API and Networking**

The **Fetch API** is a modern interface to make network requests, replacing older techniques like `XMLHttpRequest`.

```js
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data));
```

When you use `fetch`:

1. The call is passed to the **browser’s networking layer**.
2. The request is sent asynchronously.
3. The browser receives the response.
4. The response is handed back to the JavaScript engine via a **Promise**, and the callback is queued.
5. Once the event loop is ready, the callback is executed.

The browser ensures the UI remains responsive while this network I/O is happening in the background.

### 8. **Rendering Updates and Repaints**

When JavaScript modifies the DOM (e.g., changing text, styles, adding elements), the browser may need to **recalculate styles**, **re-layout elements**, and **repaint** the screen. This is expensive, so browsers optimize this process using techniques like **debouncing**, **reflows batching**, and **GPU acceleration**.

### React in the context of browsers:

React is **not part of the browser**. It runs **within** the browser’s JavaScript engine after your scripts are loaded. So once your React app's JS bundle is fetched by the browser and executed, React starts doing its job.

### **React Interacts with the DOM — via the Virtual DOM**

- React doesn't directly touch the **real DOM** every time.
- It creates a **Virtual DOM**, which is just a lightweight JavaScript object representation of what the DOM should look like.
- When state or props change, React:
    - Updates the Virtual DOM.
    - Diffs the new Virtual DOM with the previous one.
    - Calculates the **minimum set of changes**.
    - Efficiently applies those changes to the **real DOM** using `document.createElement`, `appendChild`, etc.


### React callbacks handling:

- React wraps the browser’s native event system with its own **Synthetic Event system** (a cross-browser wrapper).
    
- When you write:
```js
<button onClick={handleClick}>Click Me</button>
```

React is:

- Attaching a single event listener at the root (`document`).
- Internally managing which component should respond to the event using a system like **event delegation**.
- Running your handler (`handleClick`) in the event loop as part of normal JS execution.

#### React and rendering lifecycle:

- React renders the UI by calling component functions (or classes).
- Each render:
    - Produces a new Virtual DOM tree.
    - Gets diffed.
    - Updates the real DOM via the browser.
- React batches updates and minimizes layout thrashing, just like browsers optimize paint/layout.


#### JSDOM:

**JSDOM** is a **Node.js-based JavaScript implementation of the DOM and HTML standards**, created to simulate a browser-like environment _outside of a real browser_ — specifically **in Node.js**.

- Testing frameworks (like Jest)
- Server-side rendering (SSR)

It mimics the **DOM**, not the **browser**.

| Feature                | **Real Browser**                                                     | **JSDOM**                                                   |
| ---------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Environment**        | Runs in a full GUI browser (Chrome, Firefox, etc.)                   | Runs in Node.js (CLI, no GUI)                               |
| **DOM Implementation** | Native, full-featured DOM (C++ bindings, highly optimized)           | Pure JavaScript mock of DOM                                 |
| **Rendering Engine**   | Has layout, paint, and compositing (renders pixels to screen)        | **No rendering** — DOM exists in memory only                |
| **CSS Support**        | Parses, applies, and renders CSS                                     | Parses some CSS, but **does not render** or compute styles  |
| **Web APIs**           | Full access (Fetch, WebSockets, Service Workers, localStorage, etc.) | Limited or missing (you often need to polyfill)             |
| **JavaScript Engine**  | V8, SpiderMonkey, etc.                                               | Runs in Node.js (also uses V8, but outside browser context) |
| **Events**             | Real user input events (click, mouse, touch, keyboard)               | Synthetic only; you must simulate events manually           |
| **Security Model**     | Sandboxed, has CSP, cross-origin rules                               | No real security model (since it’s not user-facing)         |
| **Plugins/Extensions** | Supports DevTools, extensions, etc.                                  | None — it's headless, pure JS                               |

## JSDOM:

**JSDOM** is a JavaScript-based implementation of the **DOM and browser APIs**, designed to simulate a web page environment **inside Node.js** (which normally doesn’t have `window`, `document`, or any browser-specific APIs). It allows developers to use HTML and JavaScript like they would in a browser, but in a **non-visual, scriptable, headless environment**. Created by the developers behind the **jsdom** package (maintained by the `jsdom` GitHub project), it's widely used for **testing**, **server-side rendering**, and **HTML parsing or manipulation** tasks.

JSDOM can replicate any of the core browser behaviour related to html and the DOM.

```js
const { JSDOM } = require("jsdom");
const dom = new JSDOM(`<html><body><div id="app"></div></body></html>`);
const document = dom.window.document;

const el = document.createElement('p');
el.textContent = 'Hello World';
document.getElementById('app').appendChild(el);
```

JSDOM can execute inline `<script>` tags if configured with `runScripts: "dangerously"`. This lets you simulate DOM manipulation from within a loaded HTML page.

```js
new JSDOM(`<script>document.body.innerHTML = "<h1>Hi</h1>";</script>`, {
  runScripts: "dangerously"
});
```

You can create and dispatch events like `click`, `input`, etc. — useful for testing interactions in libraries like React or jQuery.

```js
const button = document.createElement('button');
button.addEventListener('click', () => console.log('Clicked!'));
button.click(); // logs: Clicked!
```

JSDOM does no actual rendering. No CSSDom and has limited web apis.

JSDOM uses:

1. Unit and component testing.Frameworks like **React Testing Library** or **Enzyme** rely on JSDOM to simulate the DOM.
```js
import { render, fireEvent } from '@testing-library/react';
import MyButton from './MyButton';

test('clicking the button triggers action', () => {
  const { getByText } = render(<MyButton />);
  fireEvent.click(getByText('Click me'));
  expect(...).toBe(...);
});
```

2. In some cases, you need to **render HTML on the server** before sending it to the browser (e.g., for SEO). JSDOM is sometimes used to **simulate a DOM** so components that rely on `window` or `document` don't crash during SSR.

### Client vs Server side rendering:

**Rendering** refers to the process of turning code (HTML, CSS, JS) into a visible **user interface** in the browser.

Depending on where this process happens — the **server** or the **client (browser)** — we have two main strategies:

- **Server-Side Rendering (SSR)**:SSR means the server generates the **full HTML** for a page **before** sending it to the browser. The browser then "hydrates" it (attaches JS to make it interactive).

- **Client-Side Rendering (CSR)**:CSR means the **initial HTML** sent from the server is **minimal or empty**, and the **browser (client)** is responsible for downloading JavaScript, running it, and then rendering the full UI.

## Basics of dom elements:

### Input elements:

A basic input element in dom holds info like

```js
<input type="text" value="hello" placeholder="Your name" />
```

- When you listen to events like `onChange` or `onClick`, React passes a **synthetic event**.
- `event.target` is the DOM element that triggered the event (the actual input or button).
- For example:

```js
function handleChange(event) {
  console.log(event.target); // The actual <input> element
  console.log(event.target.value); // The current value inside the input
}
```

| Property               | Description                                | Example                |
| ---------------------- | ------------------------------------------ | ---------------------- |
| `event.target`         | The DOM node where the event originated    | `<input ... />`        |
| `event.target.value`   | Current value of input (text inside field) | `'hello'`              |
| `event.target.name`    | The `name` attribute if provided           | `'username'`           |
| `event.target.checked` | For checkboxes/radios: boolean if checked  | `true` / `false`       |
| `event.target.type`    | The input type (`text`, `checkbox`, etc.)  | `'text'`, `'checkbox'` |
| `event.target.id`      | The id attribute if set                    | `'input-id'`           |

eg with fireEvent api:

```js
fireEvent.change(inputElement, { target: { value: 'new value' } });
```

```js
<button onClick={(e) => {
  console.log(e.target.type);  // "submit"
  console.log(e.target.id);    // "submit-btn"
}}>
  Submit
</button>
```

Full event object for input change

```js
{
  target: {
    value: 'some text',       // current input value
    name: 'email',            // name attribute if set
    type: 'text',             // type of input
    checked: undefined,       // undefined for text inputs
    id: 'email-input',        // id attribute if set
    // plus many other DOM element properties/methods...
  },
  // other event properties (stopPropagation, preventDefault, etc.)
}
```