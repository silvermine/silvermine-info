# Silvermine Coding Standards Specific to ES6+ / TypeScript

## Variable Declarations

The keyword `var` should be avoided in all new code that targets ES6+. Instead, `let`
and `const` should always be used. If the variable will not be changed, always use
`const`.

Consecutive variable declarations should be grouped in one statement. However, multiple
statements can be used if there is other code between them. The exception to this is
multi-line `const` declarations. If a `const` declaration spans multiple lines, it must
appear in a separate declaration. `let` declarations should never span multiple lines.

Example:

```javascript
const SOME_OBJ = {
   a: 1,
   b: 2,
};

const SOME_STR = 'something',
      SOME_NUM = 2;

let someOtherString = 'something else',
    someOtherNumber = 3,
    arr1 = [ 1, 2, 3, 4 ],
    someChangeableObj;

someChangeableObj = {
   x: 1,
   y: 2,
};

let myTotal = 0;

for (let x of arr1) {
   myTotal += x;
}
```


## ES6 Array Prototype Functions

At this time, we prefer the use of [Underscore.js](https://underscorejs.org/) over the
array helper functions added in ES6 (i.e. `includes`, `find`, `keys`, `entries`, etc.). We
take this stance because we develop JS apps both for Node.js and many browsers, including
quite a few legacy browsers that do not support the full range of array functionality.
Thus, rather than making developers have to stop and think about which functions are
available with each context switch, we just default to using Underscore.js so that it's
guaranteed to work in any environment we're targeting.

The exception is the `for ... of` loop structure since TypeScript transpiles it down to
ES5-compatible code.

Example:

```javascript
// use
let a = _.find(arr1, (x) => { return x > 1; });

// instead of
let a = arr1.find((x) => { return x > 1; });
```


## Template Literals

Template literals are preferred, but not enforced. The goal is, as always, to
make the code as readable as possible. If a template literal makes the code
_less_ readable, string concatenation should be used instead. For example:
`` `.${fileName}` `` is less readable than `'.' + fileName`.

The exception to the above is multi-line template literals. These should never
be used, because of the negative effect on indentation and readability.

For example, the following would be necessary to ensure no leading spaces on
the second line, and is not allowed:

Example:

```javascript
// This odd indentation is necessary to ensure that there are no leading spaces on
// the second line, and should be avoided.
function myFunction() {
  let str;

  str = `First Line
second line`;
}

// Use this instead
function myFunction() {
   let str;

   str = 'First Line\n';
   str = str + 'second line';
}
```


## async / await

**Allowed Sparingly**

We are still forming an opinion on the use of these operators. For now, we have
decided to use them sparingly and prefer the use of Promises.

The async/await style can easily lead to unintentionally synchronous code.

Example:

```javascript
async function foo() {
   let a = await asyncFunc1(),
       b = await asyncFunc2();

   return a + b;
}
```

The above code would wait for `asyncFunc1` to return before running
`asyncFunc2`. A better approach would be:

```javascript
function foo() {
   return Promise.all([ asyncFunc1(), asyncFunc2() ])
      .then(([ a, b ]) => { return a + b; });
}
```

This example would be more performant, as `asyncFunc1` and `asyncFunc2` are run
concurrently. This intricacy is easily lost in reading the `async/await` example.


## Destructuring

Basic object and array destructuring is permitted. However, some of the more
advanced destructuring techniques are difficult to read and thus not allowed.


### Basic Object/Array Destructuring

**Allowed**

Example:

```javascript
let rect = { x: 0, y: 10, width: 15, height: 20 },
    { x, y } = rect;

console.log(x, y); // 0, 10

let arr1 = [ 1, 2, 3, 4 ],
    [ a, b, c, d ] = arr1;

console.log(a, b, c, d); // 1, 2, 3, 4
```


### Array Destructuring With Rest

**Allowed**

Example:

```javascript
let arr1 = [ 1, 2, 3, 4 ],
    [ a, b, ...arr2 ] = arr1;

console.log(a, b, arr2); // 1, 2, [ 3, 4 ]
```


### Object Destructuring With Rest

**Not Allowed**

Reason: Risky. What if the object has more properties than you were expecting?

Example:

```javascript
let rect = { x: 0, y: 10, width: 15, height: 20, somethingIForgotAbout: 'oops' },
    { x, y, ...dimensions } = rect;

console.log(dimensions); // { width: 15, height: 20, somethingIForgotAbout: 'oops' }
```

### Array Destructuring With Ignores

**Not Allowed**

Reason: Readability is negatively impacted; it's easy to miss the ignored value.

Example:

```javascript
Promise.all([ p1, p2 ]).then(([ , r2 ]) => { console.log(r2); });
```


### Renaming While Destructuring

**Not Allowed**

Reason: The function of the code is not readily apparent.

Example:

```javascript
const obj = { someProperty: 'some value'},
      { someProperty: someVariable } = obj;

console.log(someVariable); // 'some value'
```


### Deep Data Destructuring

**Not Allowed**

Reason: It is not very readable and it does not do empty/null/undefined checking.

Example:

```javascript
let foo = { bar: { bas: 123 } },
    { bar: { bas } } = foo, // Effectively let bas = foo.bar.bas;
    { notThere: { baz } } = foo; // Throws an error because notThere is undefined
```


## Rest Parameters

**Allowed, Encouraged**

We prefer using the rest operator in function parameters rather than accessing
the `arguments` variable when dealing with dynamic parameters.

Example:

```javascript
function myFunction(...baz) {
   // This is preferred over console.log(arguments)
   console.log(baz);
}

myFunction('a', 'b', 'c'); // [ 'a', 'b', 'c' ]
```


## Spread

**Allowed**

Example:

```javascript
let foo = [ 1, 2, 3, 4 ],
    bar = [ ...foo, 5, 6 ];

console.log(bar); // [ 1, 2, 3, 4, 5, 6 ]

const obj1 = { x: 0, y: 1 },
      obj2 = { ...obj1, width: 10, height: 2};

console.log(obj2); // { x: 0, y: 1, width: 10, height: 2 }
```


## Arrow Functions

The use of arrow functions is encouraged where appropriate.

Arrow functions must have parentheses around the parameters.

Example:

```javascript
// valid
let square = (a) => { return a * a; };

// invalid
let square = a => { return a * a; };
```

### Implicit Returns

**Not Allowed**

Reason: We always favor explicit syntax in our codebase. Banning implicit returns makes it
more readily apparent that the function is returning a value and less likely that someone
inadvertantly changes a return type by performing an operation and forgetting that the
operation is automatically returned if no curly braces surround the statement.

Example:

```javascript
// valid
let square = (a) => { return a * a; };

// invalid because the result of `a * a` is implicitly returned
let square = (a) => a * a;

// valid (and the curly braces make it so that nothing is returned)
someList.forEach((n) => { console.log(n); });

// invalid (because the result of console.log is implicitly returned)
someList.forEach((n) => console.log(n));
```


### Whitespace

There should be a space before and after the arrow.

Example:

```javascript
// valid
let square = (a) => { return a * a; };

// invalid
let square = (a)=> { return a * a; };
```


### Arrow Functions as Class Properties

Arrow functions should not be used as TypeScript class properties. Using arrow functions
as class properties leads to several hidden dangers with regards to changing calling
context. In addition, subclasses cannot make use of `super` when overriding arrow function
properties.

The following would be **invalid**:

Example:

```typescript
class Adder {
    constructor(public a: number) {}

    add = (b: number): number => {
        return this.a + b;
    }
}
```


## Iterators and Generators

**Not Allowed**

Example:

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)


## Default Parameters

**Allowed**

Example:

```javascript
function multiply(a, b = 5) {
   return a * b;
}

console.log(multiply(2)); // 10
console.log(multiply(2, 2)); // 4
```


## Types

In TypeScript, we disallow all implicit `any`s (see
[TypeScript Deep Dive](https://basarat.gitbooks.io/typescript/docs/options/noImplicitAny.html)).

In addition, if a variable is part of the API of the function/module (i.e. it is exported
or returned), it must have an **explicit** type. Also, the return types of functions
should be explicitly defined. This will make a change to the API readily apparent to any
developer changing the type of the variable in the future.

Example:

```typescript
export const a = '1';

function square(x: number) {
   return x * x;
}
```

Although `a` in the above code has an implicit type of `string` and will pass linting and
compiler checks, this declaration would not be allowed in our codebase because `a` is
exported. This poses a potential problem, since a developer may change the value of `a`,
for example to the **number** `1`, implicitly changing its type and the public API of this
module/library, which may not be clear to the developer. By forcing an explicit type on
the variable declaration it adds one more warning to the developer that he is making a
potentially dangerous change.

Similarly, the return type of the `square` function is implicitly `number` because the
return type declaration is left off. If the developer does something later that changes
this function's returned type, the API of this function may change without the developer
realizing it. If the return type is explicitly specified, however, the developer will be
made immediately aware that the public API of this function is changing.

Instead, the following should be used:

```typescript
export const a: string = '1';

function square(x: number): number {
   return x * x;
}
```


### General Types

Never use the types `Number`, `String`, `Boolean`, or `Object`. Instead, use the lower
case, primative version.

Example:

```typescript
// invalid
let s: String = 'something';

// valid
let s: string = 'something';
```

### Whitespace

All type declarations should be written with no space between the variable name and the
colon, but one space between the colon and the type.

Example:

```typescript
// valid
let x: string = 'a string';

// invalid
let x : string = 'a string',
    y :string = 'a string',
    z:string = 'a string';
```


## TypeScript Parameter Properties

**Allowed For Private Properties**

Example:

```typescript
// valid
class SomeClass {
   public x: number;

   constructor(x: number, private _y: number) {
      this.x = x;
   }

   public add() {
      return this.x + this._y;
   }
}

// invalid
class SomeClass {
   constructor(public x: number, private _y: number) {}

   public add() {
      return this.x + this._y;
   }
}
```
