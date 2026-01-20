
---

**React Hooks** as a way to add **state and lifecycle features** to **functional components**. All hooks start with use and they must be called on the top level of functional component.  They can't work inside nested functions etc.
### useState:

With `useState`, you can declare state variables in a functional component and update their values without needing to convert the component to a class.

```jsx
const [state, setState] = useState(initialState);
```

- **`state`**: This is the current value of the state variable. It represents the state value at any given point in time.
- **`setState`**: This is a function used to update the state. You will call this function to update the state whenever you want to change its value.
- **`initialState`**: This is the **initial value** of the state. It can be any data type: number, string, object, array, etc. The `useState` hook will initialize the state variable with this value on the first render.

- **initial Render**: When the component first renders, the `useState` hook returns the initial value (i.e., the value provided as `initialState`).
- **State Update**: Calling the `setState` function triggers a re-render of the component, and React will update the value of `state` to the new value.
    
- **Subsequent Renders**: After the state is updated, React remembers the updated state and uses it in the subsequent renders until the state changes again.

### useEffect:

It allows you to **perform side effects** in **functional components**. Side effects include operations like:

- Fetching data from an API
- Subscribing to external services
- Manually modifying the DOM
- Setting up or cleaning up event listeners
- Updating the browser’s title, etc.

```jsx
useEffect(() => {
  // Code for the side effect
}, [dependencies]);
```

- **First Argument (`() => {}`)**: The function that contains the **side effect** code. This function is executed **after** the component renders.
    
- **Second Argument (`[dependencies]`)**: An array of values that React watches for changes. This array determines when the effect should be executed:
    - If no array is provided, the effect runs **after every render**.
    - If an empty array (`[]`) is provided, the effect runs only **once**, after the first render (like `componentDidMount` in class components).
    - If an array with variables is provided, the effect will run when **any** of those variables change.

Some side effects (e.g., subscriptions, event listeners) need to be cleaned up when a component unmounts or when certain dependencies change. To handle this, you can return a **cleanup function** inside `useEffect`.

### useContext:

The `useContext` hook is used to **access the value of a React context** in a functional component. It allows you to read from a context that is **provided higher in the component tree** without having to pass the context value explicitly through props. This is especially useful in large applications where prop drilling (passing data through multiple levels of components) can become cumbersome.

1. **Context** is a feature in React that allows you to share **values** (like themes, authentication status, language preferences, etc.) across the component tree without having to pass props manually at every level.
2. - It provides a **global way to pass data** through the component tree without having to prop-drill.

The **Context API** consists of two key components:

1. **`React.createContext()`**: Used to create a context.
2. **`Provider`**: Wraps your component tree and provides the context value.
3. **`useContext`**: Reads the current value of the context.

```jsx
import React, { createContext, useState } from 'react';

// Create the Context
const ThemeContext = createContext();

// Create a provider component
function App({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme((prevTheme) => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export { App, ThemeContext };
```

```jsx
import React, { useContext } from 'react';
import { ThemeContext } from './ThemeProvider';

function ThemedComponent() {
  const { theme, toggleTheme } = useContext(ThemeContext); // accepting themecontext

  return (
    <div style={{ backgroundColor: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff' }}>
      <h1>The current theme is {theme}</h1>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
}

export default ThemedComponent;
```

Another example

```jsx
const moods = {
	happy:'happy mood',
	sad:'sad mood'
}

const MoodContext = createContext(moods);

function App(props){
	
	return (
		<MoodContext.Provider value={moods.happy}>
			<ChildComponent/> // any child compoent can access moods object now
		</MoodContext.Provider>
	);
}

```

```
function ChildComponent(){
	const mood = useContext(MoodContext);
	return <p>{mood}</p>
}
```

###### Context API:

The **Context API** in React provides a way to share values across components, without having to manually pass props down through every level of the component tree (a practice known as "prop drilling"). It allows you to share data such as user authentication status, themes, language preferences, or other global settings with any component in the app, no matter how deep the component is in the tree.

The React Context API involves three main concepts:

1. **`React.createContext()`**: Creates a context object that holds the value you want to share.
2. **`Provider`**: A component that makes the context value available to all its descendants.
3. **`Consumer`**: A component (or hook) that subscribes to the context and uses the context value.

first we craete context:

```jsx
import React, { createContext, useState } from 'react';

// Create a Context object
const MyContext = createContext();
```

- **`MyContext`**: This is the context object that will hold the shared value.
- The `createContext()` function returns an object with two main properties:
    - `Provider`: A component that is used to **provide** the context value to the tree.
    - `Consumer`: A component that **consumes** the context value.

The `Provider` component is used to make the context value available to all components inside it. You typically use it at a higher level in the component tree, often at the root level of your app, so all the nested components can access the context.

```jsx
import React, { useState } from 'react';
import { MyContext } from './MyContext';
import SomeComponent from './SomeComponent';

function App() {
  const [value, setValue] = useState('Hello, World!');

  return (
    // The Provider makes the `value` available to all child components
    <MyContext.Provider value={value}>
      <SomeComponent />
    </MyContext.Provider>
  );
}

export default App;
```

You can access the context value using the `useContext` hook in functional components.

```jsx
import React, { useContext } from 'react';
import { MyContext } from './MyContext';

function SomeComponent() {
  const value = useContext(MyContext);  // Accessing the context value

  return <h1>{value}</h1>;
}

export default SomeComponent;
```

Before the introduction of hooks, `React.createContext()` provided a **`Consumer`** component to access context in class components or functional components that do not use hooks.

```jsx
import React from 'react';
import { MyContext } from './MyContext';

function SomeComponent() {
  return (
    <MyContext.Consumer>
      {value => <h1>{value}</h1>}
    </MyContext.Consumer>
  );
}

export default SomeComponent;
```
### useRef

The `useRef` hook in React is used to persist values across renders without causing a re-render. It provides a way to **reference** DOM elements or store mutable values that don't trigger re-renders when changed. This makes it particularly useful for managing **focus**, **animations**, or **storing previous values** without affecting the component's lifecycle or causing unnecessary updates to the UI.

```jsx
const myRef = useRef(initialValue);
```

##### What `useRef` Does:

1. **References DOM Elements**: It can hold a reference to a DOM element (like an input, div, or canvas) so that you can interact with it directly, such as focusing on an input or measuring the size of an element.
2. **Stores Mutable Values**: It allows you to store values that **persist across renders** without causing the component to re-render when those values change. The value inside a ref persists across renders, unlike normal variables that are reinitialized with every render.

```jsx
import React, { useRef } from 'react';

function FocusInput() {
  const inputRef = useRef(null);  // Create a ref to access the input element

  const handleClick = () => {
    inputRef.current.focus();  // Focus the input field using the ref
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Click button to focus" />
      <button onClick={handleClick}>Focus the input</button>
    </div>
  );
}

export default FocusInput;
```

- **`inputRef.current`**: After the component has rendered, `inputRef.current` will refer to the DOM node for the input field. We use `.focus()` to programmatically focus on the input.
- **`ref={inputRef}`**: The `ref` attribute is used to attach the `inputRef` to the input element. This gives us direct access to the DOM node.

`useRef` can also be used to store any mutable value that you want to **persist across renders** but don't want to trigger a re-render when updated. This is especially useful for tracking previous state or values.

```jsx
import React, { useState, useEffect, useRef } from 'react';

function PreviousValue() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();  // Create a ref to store the previous count

  useEffect(() => {
    prevCountRef.current = count;  // Update the ref value after each render
  }, [count]);  // Run this effect whenever `count` changes

  return (
    <div>
      <h1>Current Count: {count}</h1>
      <h2>Previous Count: {prevCountRef.current}</h2>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

export default PreviousValue;
```

### useReducer:

The **Reducer pattern** in React is a design pattern commonly used to manage and update complex state logic, especially when dealing with state transitions that involve multiple actions or a more complex state structure. The reducer pattern is often implemented using the `useReducer` hook in React, though it can also be used outside of React as a general JavaScript pattern.

A **reducer** is a pure function that takes two arguments:

1. **The current state** of the application (or part of the state).
2. **An action** that describes what happened.

The reducer function then returns a **new state** based on the current state and action. The idea is that the state is always **immutable**: it cannot be changed directly but rather replaced with a new state.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'ACTION_TYPE':
      return {
        ...state,
        property: action.payload
      };
    default:
      return state;
  }
}
```

- **state**: The current state of your component or application.
- **action**: The object describing the action that occurred (usually contains a `type` and optional `payload`).
- **return**: A new state based on the current state and action.

useReducer is the hook that will allow to have this pattern

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- **`reducer`**: The reducer function that defines how the state changes based on actions.
- **`initialState`**: The initial value of the state when the component first renders.
- **`state`**: The current state, which is updated based on the dispatched actions.
- **`dispatch`**: A function that allows you to send actions to the reducer function, which will update the state.

#### 1. **Create the Reducer Function:**

```jsx
const initialState = { count: 0 };

function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return initialState;
    default:
      return state;
  }
}
```

**Reducer**:

- **`increment`** action increases the `count` by 1.
- **`decrement`** action decreases the `count` by 1.
- **`reset`** action resets the counter back to the initial state.

Now use it in a component

```jsx
import React, { useReducer } from 'react';

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div>
      <h1>Counter: {state.count}</h1>
      <button onClick={() => dispatch({ type: 'increment' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Decrement</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}

export default Counter;
```

**`dispatch`**: This function is used to send an action to the reducer. Each time you call `dispatch({ type: 'actionType' })`, it triggers the reducer function and updates the state accordingly.

- Initially, the `state` is `{ count: 0 }` as per the `initialState`.
- When the "Increment" button is clicked, the `dispatch({ type: 'increment' })` action is sent, which updates the state to `{ count: 1 }`.
- Similarly, clicking the "Decrement" button sends the `decrement` action to reduce the count, and the "Reset" button resets the state back to `{ count: 0 }`.

using both context api and useReducer

```jsx
import React, { createContext, useReducer } from 'react';

// Create a context
const StateContext = createContext();

// Initial state
const initialState = { count: 0 };

// Reducer function
function appReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// Context provider component
function StateProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);

  return (
    <StateContext.Provider value={{ state, dispatch }}>
      {children}
    </StateContext.Provider>
  );
}

export { StateProvider, StateContext };
```

Consume in a component:

```jsx
import React, { useContext } from 'react';
import { StateContext } from './StateProvider';

function Counter() {
  const { state, dispatch } = useContext(StateContext);

  return (
    <div>
      <h1>Count: {state.count}</h1>
      <button onClick={() => dispatch({ type: 'increment' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Decrement</button>
    </div>
  );
}

export default Counter;
```

- **Context**: React's Context API provides a way to share data (like state) across components, without passing props manually at each level.
    
- **Reducer**: A reducer function is used to manage complex state transitions. It receives the current state and an action, and returns a new state based on the action.

First, we need to create a **context** to store the state and a **reducer** to update the state.

```jsx
// reducer.js
export const initialState = { count: 0 };

export function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return initialState;
    default:
      return state;
  }
}
```

Next, we create a **context provider** that will use the `useReducer` hook to manage the state and provide the state and dispatch function to the rest of the app.

```jsx
// StateProvider.js
import React, { createContext, useReducer } from 'react';
import { counterReducer, initialState } from './reducer';

// Create the context
export const StateContext = createContext();

// Create the provider component
export function StateProvider({ children }) {
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <StateContext.Provider value={{ state, dispatch }}>
      {children}
    </StateContext.Provider>
  );
}
```

- **`StateContext`** is a Context object that will store the global state.
- **`StateProvider`** is a wrapper component that uses `useReducer` to manage the state and `dispatch` function.
- The state and `dispatch` are passed to the `Provider` so that any component in the component tree can access the global state and dispatch actions.

```jsx
// Counter.js
import React, { useContext } from 'react';
import { StateContext } from './StateProvider';

function Counter() {
  const { state, dispatch } = useContext(StateContext);

  return (
    <div>
      <h1>Counter: {state.count}</h1>
      <button onClick={() => dispatch({ type: 'increment' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Decrement</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}

export default Counter;
```

finally we can wrap the Provider

```jsx
// App.js
import React from 'react';
import { StateProvider } from './StateProvider';
import Counter from './Counter';

function App() {
  return (
    <StateProvider>
      <Counter />
    </StateProvider>
  );
}

export default App;
```

#### useMemo:

For caching expensive calls

for example

```jsx
const expensiveCount = useMemo(()=>{
	return count**2;
},[count]) // dependency here 
```
