
---

In react everything is component. Webpage can be think of as a document and the html document can be edited through an api that api is the dom. 

The **Document Object Model (DOM)** in HTML represents the structure of an HTML document as a tree of objects, where each node in the tree corresponds to a part of the document. The DOM allows programming languages (like JavaScript) to manipulate the content, structure, and styles of the web page dynamically.

Nodes in DOM are of three types:

1. Element node: Represents HTML tags
2. Text node: Represents the text content inside an element.
3. Attribute node: Represents the attributes of an element like(id,classes) etc.

It is the job of browser to load the html document from url maybe and then parse it into dom. 
DOM has various methods to manipulate the tree.

- **`document.getElementById()`**: Returns the element with the specified ID.
- **`document.getElementsByClassName()`**: Returns all elements with the specified class name.
- **`document.querySelector()`**: Returns the first element that matches a CSS selector.
- **`document.querySelectorAll()`**: Returns all elements that match a CSS selector.

JavaScript can be used to manipulate the DOM in various ways:

- **Adding Elements**: You can create new elements using `document.createElement()`, and append them using `appendChild()` or `insertBefore()`.
- **Changing Content**: You can change the inner text or HTML of elements using properties like `innerText`, `innerHTML`, and `textContent`.
- **Changing Styles**: You can modify an element’s CSS styles via the `style` property.

```js
// Creating a new element
const newDiv = document.createElement('div');
newDiv.textContent = 'This is a new div!';
document.body.appendChild(newDiv);

// Changing the background color of an element
const container = document.getElementById('container');
container.style.backgroundColor = 'lightblue';
```

You can also interact with the DOM using events. For example, you can listen for user actions like clicks, mouse movements, or key presses.

```js
// Adding a click event listener to a button
const button = document.querySelector('button');
button.addEventListener('click', function() {
  alert('Button clicked!');
});
```

### Phases of page rendering in html:

##### Loading HTML:

The first phase involves the browser requesting the HTML document from a server. This is typically done via an HTTP or HTTPS request.The browser sends a request to the server to retrieve the HTML file (this may be an initial request or a subsequent one for resources such as images, CSS, or JavaScript).
- **Download**: The browser downloads the HTML file and begins to process it as it’s being received. The browser doesn’t wait for the entire HTML file to be downloaded before starting the next phase; it starts parsing the HTML as soon as it gets the first chunk of data.
##### Parsing HTML:

Once the browser starts receiving the HTML document, it begins parsing it to construct a **Document Object Model (DOM)**. Parsing involves breaking the HTML into individual tokens (tags, text, attributes), and organizing them into a tree structure.
- **Tokenization**: The HTML content is split into tokens (tags, text nodes, attributes, etc.). These tokens are read by the HTML parser (which is usually a part of the browser’s rendering engine).
- **DOM Tree Construction**: The tokens are then used to construct a hierarchical tree called the **DOM** (Document Object Model). Each HTML tag becomes an element node, and text between tags becomes text nodes. The DOM tree represents the structure of the webpage.
- **Render Tree Construction**: While building the DOM, the browser also creates the **Render Tree**. This is a representation of how the page should be displayed, considering elements like the visible part of the document (only the elements that are part of the layout).

##### CSS Parsing:

At the same time as the HTML is being parsed, the browser also parses any linked CSS files or internal `<style>` tags. This step creates the **CSS Object Model (CSSOM)**, which is another tree structure that describes the styles for the document. 

**CSSOM Construction**: The browser constructs a CSSOM tree by associating the styles with the corresponding DOM nodes. This tree will determine how elements should look on the page (colors, fonts, layout, etc.).

##### Javascript execution:

JavaScript is typically embedded in the HTML document (inside `<script>` tags) or linked externally via `<script src="...">`. Once the browser encounters a `<script>` tag, it must **execute the JavaScript code** before it can continue processing the rest of the page.

**Blocking**: By default, JavaScript blocks the HTML parsing process. This means that if the browser encounters a `<script>` tag while parsing the HTML, it must stop parsing the HTML and execute the JavaScript.

- **Execution Context**: The browser sets up an execution context for JavaScript (which includes the call stack and memory heap) and runs the code. Any JavaScript manipulations of the DOM or CSSOM will modify the content of the webpage.
- **DOM Manipulation**: If the JavaScript modifies the DOM or CSSOM (e.g., adding/removing elements or styles), the browser will have to recalculate the layout and potentially re-render parts of the page.

##### Layout Phase (Reflow):

After parsing the DOM and CSSOM and executing JavaScript, the browser moves into the **layout phase**. This is where the browser calculates the position and size of each element on the page.

 **Reflow**: The browser computes the exact position and dimensions of all the elements on the page. This includes calculating the size of each element (height, width, margin, padding, borders), as well as the position relative to other elements. It’s often referred to as **reflow**.

 **CSS Styles Application**: The computed styles (from the CSSOM) are applied to the DOM, and the layout is created based on these rules.

##### Paint Phase:

Once the layout is computed, the next phase is **painting**. This is where the browser fills in the pixels for each element.

### React: 

Components physically are used to describe how the final view should look and we typically do it with JSX(javascript xml). 

React actually runs the component meaning react creates what is called component instance. React puts the props in these components and returns what is called element. Now at this point nothing is being inserted into the DOM we have just created the description of it. 

In **React**, an **element** is the smallest building block of the UI. It is a plain object that describes a UI component and its properties, but it is **not** the actual rendered DOM element.
- They are the **blueprints** for what will be rendered on the screen.
- React elements can represent both **native DOM elements** (like `<div>`, `<button>`, `<p>`) and **custom components** that you define.

```js
{
	type: "main",
	key: null,
	ref: null,
	"$$typeof":Symbol(react.element),
	props:{
		children:{
			type:"h1",
			key:null... // can be recursively object
		}
	}
}
```

React elements can be created using JSX or React.createElement(). 

type here can be a string refernce to an HTML component (react calls it DOM element) or it can be a refernce to Component react calls it (component element). 

Key is uniquely identify elements among siblings.

Ref: can be used to refernce an actual DOM node.  The **`ref`** is a special attribute in React that can be added to elements or class components to **get a reference** to the underlying DOM element or class instance. 

- On **DOM elements**, `ref` allows you to access the underlying HTML element (like `<input>`, `<div>`, etc.).
- On **class components**, `ref` gives you a reference to the class instance, so you can access methods and properties of the component.

`$$typeof` is actually used by react internally to have unique value for each node. Also keep in mind that we have a Symbol() symbol essentailly takes any thing and returns hash kind of object. 
Symbols can not be in json so you can not transport the symbols across web and so it acts as the protection because you can not inject the component and react will reject that.

Till now we dealt with mostly the metadata of a React element. The main part actually is props 
```js
{
  type: 'div', // The type of the element (e.g., div, h1, MyComponent)
  props: {
    // The props associated with the element
    className: 'container',
    children: ['Hello, world!'],
    onClick: [Function]
  },
  key: null, // The key prop, used for lists of elements
  ref: null, // The ref for referencing DOM or component instances
  _owner: null // The internal React owner that created the element
}
```

- This is an object containing all the **props** passed to the React element (the properties you define in JSX or through `React.createElement()`).
- **Children** is typically a special prop that contains any nested content or child elements inside the React element.

So if u have something like this

```jsx
<div id="title">
	Hello
</div>
```

its element will be like this

```js
{
	type:"div",
	key:null,
	props:{
		id:"title",
		children:"Hello",
	}
}
```

So we have component internally react creates component instance and that utlimately generates this objects called react element and understand that this object will be created for root and will contain every component inside it. This object is what we call virtual DOM. 

Now we can understand why react is so good actually to get the dynamic behaviour we can use DOM updates however these updates are really expensive as entire dom gets recalculated and then does all the repaint for entire application. Which is slow. However creating the virtual dom since its just an object is really easy and fast. 

In simple terms, the **Virtual DOM (VDOM)** is an in-memory representation of the **real DOM** (the Document Object Model) of a web page. It is a lightweight copy of the actual DOM that React uses to determine what has changed in the UI and efficiently update only the parts of the actual DOM that need to be changed.

So when the user wants to see some changes they do the change to VDOM nad then from there we can figure out how to move forward.

#### Reconciliation:

**Reconciliation** is the process by which React updates the **real DOM** to match the **virtual DOM**. React needs to figure out which parts of the **DOM** need to be updated. Instead of re-rendering the entire DOM, React uses an efficient process to only update the parts of the DOM that have actually changed.

React uses a **diffing algorithm** to compare the **previous virtual DOM** with the **new virtual DOM** to detect what has changed.

Now how exactly it happens when the type of node changes for example from main to div then react builds the entire tree from this root from scratch. 

So when this happens for example when user is viewing the list and then they go watch the single list item.

`<Products> -     <ViewProduct>`

The new DOM node is inserted to the DOM and components receive component will mount and componentdidMount or functional components have their hooks run appropriately. Any state associated with old tree is lost.

Now what if we are viewing new product. In that case usually the id or some value inside component changes so underlying component remains the same. Only the value changes. 

Here render will be called and diff algorithm will get the diff between old and new tree and tree will be updated donw the line.

When a component’s state or props change, React creates a **new virtual DOM** based on the updated state/props.

React compares the old virtual DOM with the new virtual DOM. This process is called **diffing**. React uses efficient algorithms to figure out the minimum number of changes required.

Based on the differences found in the diffing process, React reconciles the virtual DOM with the real DOM. It applies the necessary updates to the real DOM in the most efficient way possible.

When rendering lists of elements (e.g., using `.map()`), React uses **keys** to track each element. Keys help React efficiently match the elements between renders, making it faster to update the DOM when items in the list change.

If the type of a component changes (e.g., changing from a `<div>` to an `<h1>`), React will **destroy the old component** and **create a new one**. This is because React cannot reconcile elements of different types.

React tries to minimize the need for reordering elements. If a list of items changes and the order is the same, React can update the components in place. However, if items are added or removed, React will only apply the changes that are necessary based on the keys provided in the list.

### Rendering:

render function of ReactDOM is responsible for doing all the stuff it actually starts all the tree making and all the difffing stuff but not rendering that is done by React Fiber. It is responsible for generating the dom , generating the VDOM and actually insert it into the DOM. 

```jsx
ReactDOM.render(
	<App/>,
	document.getElementById("root")
)
```

ReactDOM actually does not comes with react library it is part of react-dom library. React's job is just to do diffing. 

### React fiber:

**React Fiber** is the re-architecture of React’s core algorithm, introduced in **React 16**, which significantly improved React's ability to handle complex updates and improve performance. The main goal of **Fiber** was to make React more **responsive**, handle **async rendering** more effectively, and enable **better scheduling** of updates.

Before **React Fiber**, React’s rendering engine used a single, synchronous call to process all updates in a **batch**, which made it hard to handle large, complex UIs or high-priority updates efficiently.

Fiber enables **incremental rendering** and allows React to break the rendering work into chunks, giving the browser a chance to **work on other tasks** (like animations, input handling, etc.) between those chunks. This helps to ensure that React can stay **responsive** even when updating large components or complex UIs.

React Fiber allows rendering to be split into units of work that can be completed incrementally. This means React can start rendering a component, then pause and yield control back to the browser before finishing the work. This way, React can handle updates **without blocking the main thread** and prevent UI freezes.

