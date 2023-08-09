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

# Terms
`Execution Context` - when the code sees a set of brackets it'll create an execution context, and adds it to the stack.  You have things like `this` and all variables, methods, etc, available in a specific context.

`Lexical Environment` - where something is written/defined

`Lexical Scope` - available data + vars where func was defined determines our available vars.  Not where the function is called (`Dynamic Scope`)

`Hoisting` -  JS engine allocating memory for variables and funcs in creation phase before it executes it (lets you run the `sing()` method before it's actually been defined in the file, like right before the function is defined).  Variables are undefined initially (partially hoisted), but functions actually get defined (fully hoisted).  The first thing the engine has to see is `var` or `function` it'll hoist(NO OTHER TERMS), but if you wrap methods in parenthesis or use something like `const` it won't get hoisted.  *This occurs every time we create a function!*

`Function Expression` - `const thing = function() {}`

`Function Declaration` - `function blan() {}`