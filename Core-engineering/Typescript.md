
---

Typescript is an statci language meaning every variable has a type and must remain same through life of variable.

Typescript can figure out the types implicitely.

```typescript
let x = 5; // number
let y:number = 5; // explicit
let z:string = 5; // error
```

By default a variable is declared with any type if not automatically inferred. Note that any means it won't have any type safety.

```typescript
let x; // any type
```

### Types category:


In **TypeScript**, a **type** is a way to describe the shape, structure, and behavior of **data**. Types allow the compiler to check that your code uses values correctly — making your code **more predictable**, **readable**, and **safe**.

## 📌 1. **Primitive Types**

These are the most basic types in TypeScript.

|Type|Example|Description|
|---|---|---|
|`string`|`"hello"`|Textual data|
|`number`|`42`, `3.14`|Any number (int or float)|
|`boolean`|`true`, `false`|Logical true/false|
|`null`|`null`|Null value|
|`undefined`|`undefined`|Uninitialized or missing value|
|`bigint`|`123n`|Large integers|
|`symbol`|`Symbol("key")`|Unique identifiers|

```ts
let name: string = "Alice";
let age: number = 30;
let isAdmin: boolean = true;
```


### Object literal

```ts
let user: { name: string; age: number } = {
  name: "John",
  age: 25
};
```

### Arrays

```ts
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b"];
```

### Tuples

Fixed-length arrays with specific types.

```ts
let point: [number, number] = [10, 20];
```

### Union (`|`): Value can be one of several types

```ts
let id: string | number;
id = 123;
id = "abc";
```

### Intersection (`&`): Combine multiple types into one

```ts
type A = { name: string };
type B = { age: number };
type C = A & B; // { name: string, age: number }
```


Custom names for any type.

```ts
type User = {
  id: number;
  name: string;
};

let user: User = {
  id: 1,
  name: "Jane"
};
```

## 👤 5. **Interfaces**

Another way to define object shapes. Preferred when extending or implementing.

```ts
interface Product {
  id: number;
  title: string;
}

const p: Product = { id: 10, title: "Book" };
```

Interfaces support **extension**:

```ts
interface Book extends Product {
  author: string;
}
```

## 🧠 6. **Literal Types**

Limit a value to exact strings, numbers, or booleans.

```ts
let direction: "left" | "right";
direction = "left"; // ✅
direction = "up";   // ❌
```

## 🚀 7. **Function Types**

Define the shape of functions.

```ts
let greet: (name: string) => string;

greet = (name) => `Hello, ${name}`;
```

## 🧬 8. **Generics**

Reusable, type-safe components or functions.

```ts
function identity<T>(value: T): T {
  return value;
}

let output = identity<number>(123);
```


## ❓ 9. **`any`, `unknown`, `never`, `void`**

|Type|Description|
|---|---|
|`any`|Opts out of type checking (avoid if possible)|
|`unknown`|Like `any`, but safer (must check type before use)|
|`never`|Represents unreachable code or functions that never return|
|`void`|Used for functions that don’t return a value|

```ts
function log(message: string): void {
  console.log(message);
}

function fail(): never {
  throw new Error("Error!");
}
```

## 🔐 10. **Enums**

Named constants.

```ts
enum Direction {
  Up, // by default we values starting from 0,1...
  Down,
  Left,
  Right
}

let dir: Direction = Direction.Left;
```

### Functions:

```typescript
function doSomething(x:number,y:string)string{
	return toString(x)+y;
}
```

Numbers , boleans and strings are primitive datatypes. We can have objects in javascript to handle the type here we form interfaces or types. Both can be used to define the shape of an object.

With some caveats:

1. Interfaces can be extended but types can not be.
2. Types support Union and intersection but interfaces don't.

```typescript
interface Person{
	name:string;
	age?:number;
}

type User = {
	name:string,
	age:number,
	isAdmin?:boolean
}
```

In both we can make any property optional by ? 

Using it 

```typescript
function doSomething(userObj:User){
	
}
```

Arrays also have types:

```typescript
function doSomething(arr:string[]){
	arr[0] = 3; // error
}
```

Union and intersection help us to accept two different things in a given type. Unions and intersection bascially define a new type

```typescript
type User = {
	age:number | string,
}

let arr:(string|number)[] = []; // array with two kind of types.
```

Intersection allows us to have fields of both the types

```typescript
type Person = User & {
	gender:'Male' | 'Female'
}
```

Note that | also works with literals which means gender can only have two values.

examples:

```typescript
type Status = "success" | "error";

type Admin = { role: "admin" };
type Guest = { role: "guest" };

type User = Admin | Guest;
```

Extending an interface:

```typescript
interface Person {
  name: string;
}

interface Employee extends Person {
  jobTitle: string;
}
```

Type can also alias complex types eg

```typescript
type ID = string | number;
type Tuple = [number, string];
```

#### Tuples:

Typescript has support for tuples as well.

```typescript
type MyTuple = [number , string , boolean?]; // tuple with 3 fields again boolean is made optional using ?
```

#### Generics:

Generics allow us to pass the type as well into an object.

```typescript
function identity<T>(value: T): T { // T can now be used inside
  return value;
} 
```

However we don't create generic types but use them. 

```typescript
const [data,setData] = useState<string>("Hi");
// however the type can be auto inferred.
```

### Basics in react:

React functional component is of type FC.  It can be ommited as well.

```typescript
import {FC} from 'react';

const MyComponent:FC = (props) =>{
	// only props.children accessible.
}
```

By default only known react prop is children. 
To get others as well we can define an interface which will extend to others as well.

```typescript
interface MyProps{
	foo:number;
	bar:string;
}

const MyComponent:FC<MyProps> = (props)=>{
	return {props.foo}
};

// one way is to type like this but then we can't use destructuring.

const MyComp = (props:{text:string})=>{
	props.text
}

// another way which will allow destructing is as follows

type CardProps = {
	text: string,
	count?: number
}

export function Card({text,count}:CardProps){
	// now we can use the text and count as normal
}

// passing count is optional

<Card test={"Hello"} count={32}/>
```

Now children type

```typescript
type CardProps = {
	children: React.ReactNode // children has type ReactNode which is anything which can rendered in react
}

export function Card({children}:CardProps){
	// now we can use children
}

`<Card>
</Card>` // passing like this means children is undefined so no error.

// but in the following line it means children is missing and is unacceptable
'<Card/>' 
```

Now setter functions also have a type eg - setData they will have a type. So if you are passing a type it needs to be done in a way suppose card takes setCount

```typescript
type CardProps = {
	setCount: React.Dispatch<React.SetStateAction<number>> // number from state
}

export function Card({setCount}:CardProps){
	
}

```

Say we also pass some other method then we also need to have types here as well

```typescript
// from earlier comp
function alertMessage(message:string)string{ // can also be inferred
	alert(message);
	// return message;
}

//  now in paased function
type CardProps = {
	alertMessage: (message:string)=>void // method signature 
}

export function Card({alertMessage}: CardProps){
	
}
```

In map statements typescript is intelligent enough to understand that we don't need to explicitely type 

```typescript
return ({users.map(user => {
		return (); // use user no explicit typing
	});
});
```

Now in child component

```typescript
type CardProps = {
	user:User
}

type User = {
	name:string,
	age:number
}

export function Card({user}:CardProps){
	return (); // use the prop
}

```

One neat way is to have a folder in src called types and define the types there and export and import them bunch a times.

Using union for styling
```typescript

type CardProps = {
	color: "red" | "blue" | "purple"
}

// mapping
const colorMap = {
	red: "bg-red-500"
	blue: ".."
}
```

Note that null can not come with any datatype we have to make it union to signify that some datatype might be null as well. This is actually important when dealing with apis 

```typescript
type User = {
	name:string,
	age:number
}

function App(){
	const [user,setUser] = useState<User | null>(null); // inital null 
	return (
		// but we can't access any of properties of user
		user.name // <- user can be null as well typescript will give an error
		user?.name // <- says that user can be null 
	)
}
```

What is ? exactly - 

- This is **optional chaining**.
- It safely checks if `user` is **not null or undefined** before accessing `.name`.
- If `user` is `null`, `user?.name` returns `undefined` instead of crashing.
- If `user` is `{ name: 'Alice', age: 22 }`, it returns `'Alice'`.

Additonally we can use loading as well

```typescript
type User = {
  name: string;
  age: number;
};

function App() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    // Simulate fetching user data
    setTimeout(() => {
      setUser({ name: "Alice", age: 30 });
      setLoading(false); // Done loading
    }, 2000); // 2 seconds delay
  }, []);

  if (loading) {
    return <p>Loading...</p>;
  }

  return (
    {!loading && user && user.name} // direclty writing user.name will give error because typescript won't know that loeading will remove this condition.
  );
}
```

### Type events in react:

 Similary we have to define type of event as well we can copy type from event object.

```typescript
const App = () =>{
	const handleClick = (e:React.MouseEvent<HTMLButtonElement,MouseEvent>){
	
	}
	
	return (
		<button onClick = {(e) => handleClick(e)}>
		CLICK
		'</button>'
	)
}
```

#### Generics:

We have already seen what the generics now we will see interesting application of that
Lets define a hook that will give us data from url

```typescript
import {useState,useEffect} from 'react';

export function useFetch(url:string){
	const [data,setData] = useState(null);
	useEffect(()=>{
		async function fetchData(){
			let response = await fetch(url);
			let jsonData = await response.json();
			setData(jsonData);	
		}
		fetchData();
	},[url]);
	return data;
}
```

While this is pretty good it does not have types. Typing with generics

```typescript
import {useState,useEffect} from 'react';

export function useFetch<T>(url:string):T | null{
	const [data,setData] = useState<T | null>(null);
	useEffect(()=>{
		async function fetchData(){
			let response = await fetch(url);
			let jsonData:T = await response.json(); // says json data should be type t
			setData(jsonData);	
		}
		fetchData();
	},[url]);
	return data;
}
```

- `useEffect` starts executing `fetchData()`
- It fetches data asynchronously
    
- After data is fetched and parsed:
    - `setData(jsonData)` is called
    - This causes the component (and the hook) to **re-run**
        
- On the next render:
    - The hook runs again
    - `useState` now holds the new `data` value
    - `return data;` returns the **actual fetched data**

Now the type can be used.

```typescript
const data = useFetch<string>("url here");
```

### TSCONFIG

```json
{
  "compilerOptions": {
    // compilation rules and settings
  },
  "include": [
    // files or directories to include
  ],
  "exclude": [
    // files or directories to ignore
  ],
  "files": [
    // optional, explicitly listed files
  ],
  "extends": "base-config.json",
  "references": [
    // for project references in monorepos
  ]
}
```

| Option                             | Description                                                                              |
| ---------------------------------- | ---------------------------------------------------------------------------------------- |
| `target`                           | Specifies the JavaScript version to compile to. E.g., `"ES5"`, `"ES6"`, `"ES2020"`       |
| `module`                           | Defines the module system. E.g., `"CommonJS"`, `"ESNext"`, `"AMD"`                       |
| `strict`                           | Enables all strict type-checking options (`true` is recommended)                         |
| `esModuleInterop`                  | Enables default import compatibility between CommonJS and ES modules                     |
| `forceConsistentCasingInFileNames` | Ensures that imports use consistent case (prevents issues on case-sensitive filesystems) |
| `moduleResolution`                 | Determines how modules are resolved. Often `"node"`                                      |
| `sourceMap`                        | Generates `.map` files for debugging (useful for tools like Chrome DevTools)             |
| `outDir`                           | Directory where compiled JavaScript files will go                                        |
| `rootDir`                          | Defines the root of your TypeScript source code                                          |
| `declaration`                      | Generates `.d.ts` files (declaration files for type definitions)                         |
| `noEmit`                           | Doesn’t output JavaScript files — just type-checking                                     |
| `skipLibCheck`                     | Skips type-checking of declaration files in `node_modules` (faster builds)               |

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "CommonJS",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "sourceMap": true,
    "skipLibCheck": true
  }
}
```