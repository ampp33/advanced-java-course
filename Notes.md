# The Javascript Engine
- JIT compilation
	- Code is first interpreted
	- *Profiler* watches code and figures out ways it can be optimized
	- If some lines of code are run more than once it passes that code onto the *compiler*
	- Replaced bytecode with machine code
	- *Knowing all this helps us write more optimized code, and avoid confusing the compiler*

## Optimization
- Things to avoid
	- `eval()`
	- arguments (use destructuring instead)
	- for in (use `Object.keys()` instead)
	- with
	- delete
- Inline caching
	- Code that executes the same function repeatedly uses this
	- Instead of looking up parameters and running the method again it just replaces the method with the value the method returns
- Hidden Classes
	- `delete` really effects this
	- To avoid deoptimization, assign all properties in a constructor and setting variables in the same order

## WebAssembly
- Standard binary executable for browsers

## Memory Leaks
- Common causes
	- Too many global variables or overly large global variables
	- Adding event listeners but never removing them
	- `setInterval()` - referencing objects in here but NEVER clear the interval will stop those referenced objects from ever being cleaned up

## Single Threaded
- Has only one callstack
- No running functions in parallel
- JS is synchronous

# Javascript Runtime
- Web API comes with the browser in its Javascript Runtime
	- `fetch`
	- `setInterval()`
	- Implemented by C++ or something similar in the Browser
	- *Asynchronous*
- Call Stack will call out to Web API for things like `setTimeout()` and the Web API will handle it in a separate thread/background
- Response from Web API will add an event to the Event Loop which notifies the Call Stack *once it's free* that there's something in the Callback Queue, and *when it's available will add it to the Call Stack*

```
Call Stack -> Web API -> Event Loop + Callback Queue -> Call Stack
```

## Cool Example
```javascript
console.log(1)
setTimeout(() => console.log(2), 0)
console.log(3)

// will output:
// 1
// 3
// 2

// This execution will happen in this order NO MATTER WHAT time we set in setInterval() because setInterval() is part of the Web API and will only ever be ran AFTER the Call Stack is empty, which won't occur until the Javascript file is completely ran and done!
```

# NodeJs
- Javascript *runtime*
![[Pasted image 20230808094638.png]]
- Doesn't have `window` but does have `global`, which includes stuff like
	- `setInterval()`
- Offers a new type of server model for handling requests
![[Pasted image 20230808094958.png|400]]
	- PHP
		- Used to create a thread for each request
		- Thread pool (however many reqs the server can accept) can max out
	- NodeJS
		- Any req comes in and gets tossed onto it's async runtime
		- Can handle as many requests as you want

# Terms and Special Methods
## Terms
`Execution Context` - when the code sees a set of brackets it'll create an execution context, and adds it to the stack.  You have things like `this` and all variables, methods, etc, available in a specific context.

`Lexical Environment` - where something is written/defined.  Can be accessed via `[[scope]]`

`Lexical Scope / Static Scope` - available data + vars where func was defined determines our available vars.  Not where the function is called (`Dynamic Scope`)

`Hoisting` -  JS engine allocating memory for variables and funcs in creation phase before it executes it (lets you run the `sing()` method before it's actually been defined in the file, like right before the function is defined).  Variables are undefined initially (partially hoisted), but functions actually get defined (fully hoisted).  The first thing the engine has to see is `var` or `function` it'll hoist(NO OTHER TERMS), but if you wrap methods in parenthesis or use something like `const` it won't get hoisted.  *This occurs every time we create a function too!*  **Avoid hoisting if we can, by not using `var`, and instead use `const` and `let`.**

`Function Expression` - `const thing = function() {}` - defined at runtime

`Function Declaration` - `function blah() {}` - defined at parse time

`arguments` - reserved variable present in all function execution contexts, and is an index based object that points to each of the arguments passed in. (ex: `marry('bob', 'sally')` would have the following `arguments` value `{0: 'bob', 1: 'sally'}`).  This is dangerous to use, was initially used, but really shouldn't be used anymore (according to the author).  Meh, this seems fine to use if it's useful and you're careful.

`Variable Environment` - where variables live in the execution context

`Scope Chain` links us to variables and things inside the parent environment

`'use strict'` was added to stop JS from trying to do weird tricks/pitfalls

`function scope` - scope inside something between brackets, like inside an `if` statement.  If you use `var` the variable defined there will be available outside of the block.  Other languages use `block scope` that wouldn't allow this variable to be accessed globally.  If you use the `let` or `const` keyword tho it'll cause `block scope` to be used.

`this` - can unexpectedly refer to the `Window` object in the browser, even inside a function, which can be odd.  This can go away with the `use strict` flag.  `this` == "who called me"?  Isn't lexically scoped, is actually dynamically scoped: *it only matters who called it*

`use strict` - Disables funky weird JS behavior.  ES6 modules use this by default.

`arrow functions` - lexically scoped

`scope` is a function based thing, `context` is more about objects (ex: what's the value of the `this` var?).  `context` relates to how and who calls a function, while `scope` relates to the visibility of the variables.

`let` - lets you use block scoping

## call()
`a()` is the same as `a.call()`, all functions have `call()` implicitly added... However, `call()` calls a method of an object, substituting another object for the current object.

## apply()
Same as `call()`,  except that `apply()` takes an array of parameters rather than individual parameters

```
// call()
obj.stuff.call(anotherObj, 1, 2, 3, 4)

// apply()
obj.stuff.call(anotherObj, [1, 2, 3, 4])
```

## bind()
Returns a new function with a certain context and parameters.  Usually used when we want a certain function to be called later on with a specific context.  Lets us specify the context and parameters to pass to the function ahead of time, and we can call the function later.  Nifty!

### bind() and currying
`bind()` lets you bind to a method without specifying all parameters, thus we can easily curry a method!

# Types
- Primitive - *single values in memory*, *have object wrappers* (primitives act as you'd expect, but have functions attached to them like an object), and *immutable*
	- Numbers
	- Boolean
	- String
	- Undefined
		- Absence of definition
	- Null (type == object - is a mistake the creator admitted)
		- There's no value there.  Absence of value.
	- Symbol('just me') (ES6)
		- Used for obj properties to ensure the property is unique
		- Won't use much
- Non-Primitive - *doesn't contain actual value directly*
	- Objects {} (which is also an array and function)

`typeof` - tells us the type of a variable

```mermaid
graph TD
	A["Object {}"]-->B["Array []"];
	A-->C["Function ()"];
```
*Arrays and functions are objects*

`typeof function == 'function'`, but underneath the hood it's an object

## Arrays
These two are effectively equivalent
```
[1,2,3]

{
	0: 1,
	1: 2,
	2: 3
}
```

`Array.isArray()` tells us if an array/object is an array

## Deep Clone
`JSON.parse(JSON.stringify(obj))` - deep clone that works, but can be CPU intensive, should probably clone a different way

## Type Coercion
The language converting a type to another type

JS has a heavy hand in this because it's dynamically typed, and JS tries to help you (sometimes to a fault)

`==` - compare with type coercion (don't use this, it's confusing)
`===` - compare without coercion

# The Two Pillars (Closures and Prototypal Inheritance)
Presenter thinks `Closures` and `Prototypes` give you superpowers in JS

## Functions are objects
JS under the hood creates a "callable object"

```js
function blah() {
	console.log('hey')
}

// is effectively the same as the following (tho this code won't work)

// 'callable object' that gives it some bonus props and methods, plus can be called via ()
blah = {
	name: 'blah', // doesn't exist for anon functions
	(): console.log('hey'),
	call(),
	apply(),
	bind()
}
```

`new Function()` - function constructor, where you pass the function text into the constructor, and you can tell it the parameter names as well

`First Class Citizens`
1. Functions can be assigned to vars and properties of objects
2. Pass functions as arguments to another function
3. Can return functions as a value from other functions

`Higher Order Functions` - Function that can take an argument as an arg, or returns a function

### Things to watch out for
- Init functions in loops
- Good to set default params on a function to avoid edge cases
## Closures
We get these because `functions` are first class citizens and `lexical scope`

Example:
```js
function a() {
	const grandpa = 'grandpa'
	return function b() {
		const father = 'father'
		return function c() {
			const son = 'son'
			console.log(`${grandpa} > ${father} > ${son}`)
		}
	}
}

const one = a()

// the fact that the b() function has access to the 'grandpa' variable even tho a() is off the stack (and its variables), THAT is a closure, the extended scope that something preserves.

// grandpa goes up into the 'closure box', aka the heap, and the gc won't clean it up because it's inside a closure (and is still referenced)

// c() will look inside the closure box to find the variables it's looking for
```

- Closures are a feature of JS
- Use `lexical scoping / statically scoped`
- Even Web API calls still can access closure data

Weird Scenario
```js
function doStuff() {
	setTimeout(function() {
		console.log(stuff)
	}, 4000)
	const stuff = 'hey'
}

doStuff() // will print 'hey' after 4 seconds, because the closure still keeps that var because it sees that it's referenced, and the Web API calls out for it
```

### Main Benefits of Closures
1. Memory efficient

```js
function bigDatabaseOperation() {
	const data = [1,2,3] // pretend this is some super heavy db call
	return function(index) {
		return data[index]
	}
}

const bigData = bigDatabaseOperation()
bigData(2) // can use this instead of hitting the database each time
// this also avoids storing the data somewhere in a storage object or polluting the global scope
```

2. Encapsulation
You can store data inside of a function/object and it can maintain state + have functions you can expose to interact with that data.

```js
function basicallyAClass() {
	let value = 0
	// return public methods and data (expose only what you want)
	return {
		inc: () => value++,
		getVal: () => value,
		reset: () => value = 0
	}
}

const counter = basicallyAClass()

counter.inc()
console.log(counter.getVal()) // 1
counter.inc()
counter.inc()
console.log(counter.getVal()) // 3
counter.reset()
console.log(counter.getVal()) // 0
```

## Prototypes
JS uses *prototypal inheritance* for creating objects

Use the `__proto__` property of the object to look up the chain of objects.

```
object (instance) __proto__
-->
object type __proto__
-->
parent object type __proto__
```

Applying a prototype to an object

```js
const animal = {
	age: 1,
	breathe() => console.log('whew!')
}

const dog = {
	bark() => console.log('woof!'),
	breathe() => console.log('pant!')
}

// this makes the dog 'extend' animal, tho the properties in 'dog' won't get overwritten (like breath())
dog.__proto__ = animal
dog.age // 1
dog.breathe() // pant!
```

This is basically just inheritance, except that the base "objects" are prototype objects

Saves memory if you have lots of the same object, since the methods all share the same method in memory.

Don't use `__proto__` though, it's bad for performance and there's better ways to inherit (in OO section of course)

<<<<<<< HEAD
Only `functions` have the `prototype` and `__proto__` attributes (except the base `Object`, that has it too - since it's a type of `Function`), and are *always* a function
- don't really use the `prototype` on the actual object, but really only use them when using constructor `Functions`

`__proto__` points toward the `prototype` object on the type we inherited from.  The current function will also have its own `prototype` object

`Object` is a `Function`, `Object` is a constructor that creates the object wrapper

`Object` can be used to create `{}` (`Object.prototype`), which `Object` updates with all the variables and whatnot that it needs
- When you create one of these objects it creates it from the `prototype`

Prototypes avoid us repeating ourselves (duplicating functions in memory)

## Scheme + Java
JS was inspired by these two languages

`Scheme` - created at MIT in the 70s, one of the first languages to have closures
- functional
- had functions as first class citizens
`Java` - was trendiest at the time
- classes were important and OO

# Object Oriented Programming
Programming Paradigm - make code easier to reason about
Bringing data and its behavior in a single location (object) makes it easier to understand
Like building a robot (a head, arm, legs, etc and you build all that together)
*Prototypes*

Data == State
Methods == Actions (manipulate state)

Paradigms
- more clear and understandable
- easy to extend (easier for more devs and as program expands)
- easy to maintain
- memory effecient
- DRY (do not repeat yourself)

Lets us make complex code more organized

Two main things in programming
- data
- behavior

# Functional Programming
Data and behavior are distinctive things and should be kept apart for clarity
Give me data and functions, and I'll return something from that processed data (pipe)
*Closures*