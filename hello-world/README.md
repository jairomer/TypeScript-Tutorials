I followed [this tutorial](https://youtu.be/d56mG7DezGs).

# Lessons

The most important lessons in this tutorial involves:
- Setting up the developer environment for using TypeScript.
- Learning about TypeScript types.

I will add my notes here for whoever needs them (probably just future me).

## Development environment

I was able to do everything the tutorial proposes only using my existing neovim setup.

If you use a setup such as mine, I recommend using Mason to download and setting up TypeScript linters.

- Setting up the TypeScript development environment.
	- We can install it via [[npm.js]] with `sudo npm i -g typescript`
	- Verify the installation with `tsc -v`
- Creating a configuration file
	- `tsc --init`
	- Flow of compilation: `src` -> compiler -> `dist`
	- Just do `tsc` and the transpilation will kickstart.
- How to debug TypeScript applications.
	- Uncommenting the `sourceMap` option in `tsconfig.json` will create map files to be used by debuggers.
	- We will use the vscode debugger, for which we will need to configure a `.vscode/launch.json` in our workspace with the following data.
	- You can use the `debugger;` instruction in your code to setup breakpoints for the node inspector.
	- To get snapshots of your heap, you can use `heapdump`, available from npm or use heapsnapshot from the node inspector.
	- You can connect your JavaScript runtime to Chrome Dev Tools and get additional insights on the execution of your code. 
	- See [How to: Heap Snapshots](https://medium.com/@wavded/how-to-heap-snapshots-aac9284d5329)
**.vscode/launch.json**
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}/src/index.ts",
            "preLaunchTask": "tsc: build - tsconfig.json",
            "outFiles": [
                "${workspaceFolder}/**/*.js"
            ]
        }
    ]
}
```

## TypeScript features

The most interesting lesson regarding types is how you can leverage them to control the flow of execution and automatically infer the data structure you are dealing with.

This, in addition of type security, is the main selling point of using TypeScript on my future front-end development code.

- JavaScript already supports some types.
	- number
	- string 
	- boolean
	- null
	- undefined
	- object
- Typescript includes the following ones.
	- any
	- unknown
	- never
	- enum
	- typle

### Primitive Types

```ts
let sales: number = 123_456_789;

// Type anotations are optional
// The type of the variable is determined on assignment.
let sales_2 = 123_456_789; 
let course: string = 'TypeScript';
let is_published: boolean = true;

// If no assignment nor anotation is made, then the variable
// shall be of type 'Any'.
let level; // Any
```

### Any

- Can represent any kind of value, used for uninitialized values.
- Using this type misses the point of using TypeScript, so its use should be minimized as much as possible.
	- You will use it when integrating with existing JavaScript projects.
	- You deactivate `strict: true` in the `tsconfig.json` file.
		- `"noImplicitAny": "false"` will deactivate errors regarding Any.

```ts
function render(document) {
	console.log(document); // This will have type Any
}

function render(document: any) {
	// I know what I am doing.
	console.log(document); // This will have type Any
}
```

### Arrays
```ts
let numbers = [1, 2, '3']; // This is valid in JS, but will return an error when annotated.

// Adding a type annotation will enforce an homogeneous
// type in the components of the Array.
let numbers_ts: number[] = [1, 2, 3]; 

let numbers_any = []; // An array of type Any, which should  be avoided.

// Anotating your containers will allow tools such as IntelliSense to
// infer the available operations as you write your application code.
let numbers_ann: number[] = [];
```

### Tuples

- Tuples are defined as aggregation of elements of different or the same type of fixed length.
- Useful when working with pairs, triplets, etc...
	- More than 3 values results in code difficult to understand, so try to avoid that.
- When transpiled to JavaScript, these will be presented as JavaScript Arrays.

```ts
let user: [number, string] = [1, 'Mosh'];

// This will yield a compilation error.
let user_2: [number, string] = [1, 'Mosh', 0];
```

### Enum

- Enums are a list of related constants.
```ts
// const small = 1;
// const medium = 2;
// const large = 3;

// PascalCase naming convention
enum Size { Small = 0, Medium = 1, Large = 3 };

// If we use const, the compiler can make assumptions and generate
// more optimized code.
const enum Size_s { Small = 's', Medium = 'm', Large = 'l' };

let mySize: Size = Size.Medium;

let mySize: Size = Size.Medium;

console.log(Size.Medium); // This will return the value of the constant.
```

### Functions
- Functions in TypeScript have typed parameters and typed return values.
	- The concrete Types can be determined by the typescript compiler.
	- But as a best practice, we should properly annotate these functions.

```ts
// Set the compiler option '"noUnusedParameters": true' to detect unused parameters in your code.
function calculateTax(income: number): number {
	return 0;
}

function calculateTax(income: number): number {
	let x; // Set the compiler option '"noUnusedLocals": true' to detect unused variables in your code.
	if (income < 50_000)
		return income * 1.2;
	return income * 1.3;
}

// Set the compiler option '"noImplicitReturns": true' to detect these functions.
function calculateTax(income: number) {
	if (income < 50_000)
		return income * 1.2;
	// undefined
}

function calculateTax(income: number): void {
	console.log(income);
}

// You can define optional parameters.
function calculateTaxPerYear(income: number, taxYear?: number): number {
	// if taxYear is undefined, 2022 will be used.
	let year = (taxYear || 2022); 
	return 0;
}

// You can define default values.
function calculateTaxPerYear(income: number, taxYear = 2022): number { return 0; }

// In TypeScript you need to provide the exact number and type of parameters.
calculateTax(10_000, 2022); // This will return an error in typescript.
```

### Objects

```ts
let employee = { id: 1 }; // This object has an 'id' property of type 'number'

// This will fail, because the shape of this object has been inferred by the compiler.
employee name = 'Mosh';


// Make sure that your code is conceptually correct, this code 
// for example makes no sense because all employees are expected to have a
// name.
let employee_2: {
	id: number,
	name?: string // Optional property in this object, you should avoid this.
} = { id: 1}

let employee_3: {
	readonly id: number, // This property cannot be modified after being defined.
	name: string,
	retire: (date: Date) => void // This is a function definition that takes in a date and returns nothing.
} = {
	id: 1,
	name: 'Mosh',
	retire: (date: Date) => {
		console.log(date);
	}
}
```

### Type Aliases

- In TypeScript, you can define classes of objects called 'Aliases' to avoid repeating yourself and defining a consistent shape for these objects that will allow us to define a custom type.

```ts
type Employee = {
	readonly id: number,
	name: string,
	retire: (date: Date) => void
}

let employee: Employee = {
	id: 1,
	name: 'Mosh',
	retire: (date: Date) => {
		console.log(date);
	}
}
```

### Union Types

- We can give a variable or function parameter more than one type.
- This will not be in your JavaScript code, it is just for your compiler.

```ts
function kgToLbs(weight: number | string): number {
	// weight can be either a number or a string.
	// You will only be able to use common methods between numbers and strings.
	// This will avoid calling to 'undefined' type functions in your code.
	if (typeof weight === 'number') {
		// This technique is called "narrowing"
		// Now you can use number specific methods and functions.
		return weight *2.2;
	} else {
		// The weight is a string, so all string-specic methods and functions
		// will be valid here.
		return parseInt(weight) * 2.2;
	}
}

kgToLbs(10);
kgToLbs('10kg');
```

### Intersection Types
- This will allow us to combine types and asume a super-set of type capabilities.

 ```ts
type Draggable = {
	drag: () => void
};

type Resizable = {
	resize: () => void
};

// An UIWdiget will be able to drag() and resize().
type UIWidget = Draggable & Resizable;
 ```

### Literal Types
- Sometimes we want to limit the values we want to assign to a variable.
- This is why we use literal types.

```ts
// This can take any number in JavaScript, but a negative quantity makes no sense.
let quantity: number; 

// We use a literal type to restrict the values that can be assigned.
let quantity_1: 50 | 100 = 100;
let quantity_2: 50 | 100 = 50;

// You can use an Alias to define the conditions.
type Quantity = 50 | 100;
let quantity_3: Quantity = 100;
```

### Nullable Types
- By default TS is very strict with `null` because it is a common source of bugs in applications.

```ts
function greet(name: string) {
	console.log(name.toUpperCase());
}

// You can set 'strictNullChecks': false" to deactivate this, but it is not recomended.
greet(null); // This will fail.

function nullable_greet(name: string | null | undefined) {
	if (name)
		console.log(name.toUpperCase());
	else
		console.log('Hola!'); // default behavior.
}

// Now you can accept nullable values.
nullable_greet(null);
nullable_greet(undefined);
```

### Optional Chaining
- While working with nullable objects, we often have to do null checks.

```ts
type Customer = {
	birthday: Date
};

function getCustomer(id: number): Customer | null | undefined {
	return id === 0 ? null : { birthday: new Date() };
}

let customer = getCustomer(0);
console.log(customer.birthday); // customer might be null, so this yields an error.

// The following check will work, but there is a better way in TypeScript to do this.
if (customer !== null && customer !== undefined)
	console.log(customer.birthday);

// Optional Property Access Operator.
// This line of code will only execute if the customer is neither null or undefined.
let customer_2 = getCustomer(1);
console.log(customer_2?.birthday); // This will work and return undefined.

// This will work if there is a customer and the customer has a birthday,
// otherwise it will return undefined.
console.log(customer_2?.birthday?.getFullYear()); 

// Optional element access operator.
//  - Used in arrays and for undefined or null functions.
// let log: any = (message: string) => console.log(message);
let log: any = null;
log('a'); // This will fail.
log?.('a'); // This will execute only on those cases in which 'log' is neither null nor undefined.
```
