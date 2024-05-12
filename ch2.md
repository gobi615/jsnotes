Chapter 1: Types
While many would claim JS doesn't have types, it does. The engine and the developer treat 42 and "42" differently, therefore they have different types. They may not be as strong as in other languages, but they exist, and the ECMAScript specification defines specific types.

A Type By Any Other Name...
Types are important to understand because converting between types (and coercion) happens all the time and it's crucial to understand types fully.

Built-in Types
Seven built-in types:

- null
- undefined
- boolean
- number
- string 
- object
- symbol (new in ES6)

All types except object are primitives.

The `typeof` operator can be used to inspect a given value, returning a string representing the type.

`null` is a special case, based on a bug that's been present for a long time, and must be checked differently:

```js
typeof null === "object"; // true

var a = null;

(!a && typeof a === "object"); // true
```

`null` is the only primitive that is "falsy" that also returns "object" from `typeof`.

`typeof` can also be used on functions:

```js
typeof function a() { /* ... */ } === "function"; // true
```

```js
typeof undefined //"undefined"
typeof true //"boolean" 
typeof 1 //"number"
typeof "str" //"string"
typeof {} //"object"
typeof Symbol() //"symbol"  
typeof function () {} //"function"

typeof null //"object" (an old JS bug)
```

Functions are a subtype of object, known as "callable objects". They can have properties because they are objects.

Functions also have a `length` property corresponding to the number of formal parameters they are declared with.

```js
x = function () {}
x.length //0

y = function (a, b, c) {}
y.length //3
```

Arrays, however, are just objects, though similar to functions they may be considered a "subtype" of object.

```js
typeof typeof 42; 
// returns "string"
```

Values as Types
Variables do not have types (can hold any value at any time). Values have types.

JavaScript does not have type enforcement (engine doesn't care if the type changes during runtime).

Values can be created from other values of different types through coercion.

undefined vs "undeclared"
Variables with no value have the `undefined` value. `typeof` will return "undefined" for them.

Variables that have not yet been declared are not the same as variables that have been declared but have no value yet. Attempting to access undeclared variables results in a `ReferenceError`, while attempting to access declared variables with no value returns `undefined`.

Despite this, calling `typeof` on an undeclared variable will also return "undefined" (but this proves useful at times).

typeof Undeclared
Since multiple script files can all load variables into the global namespace, having `typeof` return "undefined" for undeclared variables can be a safeguard. Calls to global variables will not throw `ReferenceError`s if `typeof` is used to check for their existence, but would if the check is made against the variable itself:

```js
var a;
a; // undefined
b; // ReferenceError: b is not defined

var a;
typeof a; // "undefined"
typeof b; // "undefined"

// throws an error
if (DEBUG) {
console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
console.log( "Debugging is starting" );
}
```

An alternative to using `typeof` to check global variables is to check against `window.DEBUG`, since all global variables are a property of `window` in a browser. This will not work in other environments including server-side Node.js, however.

Chapter 2: Values

1. JS arrays can hold values of multiple different types.

2. Using `delete` on an array value will remove that slot from the array, but even if you remove the final element, it does not update the `length` property, so be careful!

Be careful about creating "sparse" arrays (leaving or creating empty/missing slots):

```js
var a = [ ];

a[0] = 1;
// no `a[1]` slot set here
a[2] = [ 3 ];

a[1];  // undefined

a.length; // 3
```

While that works, it can lead to some confusing behavior with the "empty slots" you leave in between. While the slot appears to have the `undefined` value in it, it will not behave the same as if the slot is explicitly set (`a[1] = undefined`).

3. Arrays are numerically indexed (as you'd expect), but the tricky thing is that they also are objects that can have string keys/properties added to them (but which don't count toward the `length` of the array):

```js
var a = [ ];

a[0] = 1;
a["foobar"] = 2;

a.length;  // 1
a["foobar"]; // 2
a.foobar;  // 2
```

However, a gotcha to be aware of is that if a string value intended as a key can be coerced to a standard base-10 number, then it is assumed that you wanted to use it as a number index rather than as a string key!

```js
var a = [ ];

a["13"] = 42;

a.length; // 14
```

Generally, it's not a great idea to add string keys/properties to arrays. Use objects for holding values in keys/properties, and save arrays for strictly numerically indexed values.

4. Reversing a string (strings are immutable so we have to return a new string instead of making changes in itself)

Another workaround (aka hack) is to convert the string into an array, perform the desired operation, then convert it back to a string.

```js
var a = "foo";
var b = ["f", "o", "o"];

c = a.toUpperCase(); // returns new string which is stored in c
a === c;  // false
a;   // "foo"
c;   // "FOO"

b.push("!"); // in-place modifier
b;   // ["f", "O", "o", "!"]
```

Many of the array methods we can actually use also on strings.

```js
var e = Array.prototype.some.call(a, (x) => x === "f"); // true
var e = Array.prototype.some.call(a, (x) => x === "e"); // false

var f = Array.prototype.find.call(a, (x) => x === "f"); // "f"
var f = Array.prototype.find.call(a, (x) => x === "e"); // undefined
```

```js
var c = a
  // split `a` into an array of characters
  .split("")
  // reverse the array of characters
  .reverse()
  // join the array of characters back to a string
  .join("");

c; // "oof"
```

If that feels ugly, it is. Nevertheless, it works for simple strings, so if you need something quick-n-dirty, often such an approach gets the job done.

Warning: Be careful! This approach doesn't work for strings with complex (unicode) characters in them (astral symbols, multibyte characters, etc.).

Immutable strings:

```js
a = "hello";
a[2] = "Z";
a //"hello"

a.toUpperCase(); //"HELLO"
a //"hello"
```

Mutable arrays:

```js
b = ["h", "e", "l", "l", "o"];
b[2] = "Z";
b //["h", "e", "Z", "l", "o"]

b.push("!")
b //["h", "e", "Z", "l", "o", "!"]
```

5. Number methods:

```js
var a = 42.59;

a.toFixed(0); // "43" 
a.toFixed(1); // "42.6"
a.toFixed(2); // "42.59"
a.toFixed(3); // "42.590"
a.toFixed(4); // "42.5900"

42.001.toPrecision(2) //"42"
42.001.toPrecision(3) //"42.0"
42.001.toPrecision(4) //"42.00"
42.001.toPrecision(5) //"42.001"
42.001.toPrecision(6) //"42.0010"

/*
The number after the e represents the number of places the 
decimal point was moved to either the left (+) or right (-).

The decimal moves until it reaches a number between 1 and 10.

Exponential notation is used for simplifying very large or very
small numbers.
*/
42.001.toExponential() //"4.2001e+1"
1000..toExponential() //"1e+3"
.001.toExponential() //"1e-3"
312000000000..toExponential() //"3.12e+11"
```

Numbers are generally expressed as base-10 decimal literals. JavaScript uses "floating-point" numbers.

```js
var a = 42;
var b = 42.3;

var a = 0.42;
var b = .42;

var a = 42.0;
var b = 42.;

var a = 42.300;
var b = 42.0;

// numbers are output with trailing fractionals removed
a; // 42.3
b; // 42

// very large and small numbers will be output in exponent form
var a = 5E10; 
a;     // 50000000000
a.toExponential(); // "5e+10"

var b = a * a;
b;     // 2.5e+21

var c = 1 / a;  
c;     // 2e-11
```

We can use `Number.prototype` functions directly on the number literal.

```js
// invalid syntax:
42.toFixed(3); // SyntaxError because . is a part of 42. literal which is valid and then there is no . property operator present

// these are all valid:
(42).toFixed(3);  // "42.000"
0.42.toFixed(3);  // "0.420"
42..toFixed(3);   // "42.000"
```

We can represent numbers also as binary, octal, hexadecimal. Lowercase predicates are preferred: `0b`, `0o`, `0x`.

```js
0b11110011; // binary for: 243
0o363;   // octal for: 243  
0xf3;   // hexadecimal for: 243
```

Small Decimal Values

```js
0.1 + 0.2 === 0.3; // false
```

The representation for `0.1` and `0.2` in binary floating-point are not exact. The sum of those two values is `0.30000000000000004`. For this type of calculation it's best to use a big numbers library.

6. Another quirk

Decimal imprecision
In JavaScript, `0.1 + 0.2 === 0.3` returns `false`. The addition actually equals `0.30000000000000004`. This is a side effect of using binary floating point numbers, which sometimes lack precision. Binary floating points are also used in other languages, not just JavaScript.

ES6 introduces `Number.EPSILON`, which allows us to check if the difference between the actual result and the expected result falls within a rounding error tolerance of `2.220446049250313e-16`.

`(0.30000000000000004 — 0.3) < Number.EPSILON` returns `true`.

```js
(0.30000000000000004 — 0.3) < Number.EPSILON
```

7. Safe Integer Ranges

Because of how numbers are represented, there is a range of "safe" values for the whole number "integers", and it's significantly less than `Number.MAX_VALUE`.

The maximum integer that can "safely" be represented (that is, there's a guarantee that the requested value is actually representable unambiguously) is `2^53 - 1`, which is `9007199254740991`. If you insert your commas, you'll see that this is just over 9 quadrillion. So that's pretty darn big for numbers to range up to.

This value is actually automatically predefined in ES6, as `Number.MAX_SAFE_INTEGER`. Unsurprisingly, there's a minimum value, `-9007199254740991`, and it's defined in ES6 as `Number.MIN_SAFE_INTEGER`.

8. Testing for Integers

To test if a value is an integer, you can use the ES6-specified `Number.isInteger(..)`:

```js
Number.isInteger(42);   // true
Number.isInteger(42.000); // true
Number.isInteger(42.3);  // false
```

To test if a value is a safe integer, use the ES6-specified `Number.isSafeInteger(..)`:

```js
Number.isSafeInteger(Number.MAX_SAFE_INTEGER); // true
Number.isSafeInteger(Math.pow(2, 53));    // false
Number.isSafeInteger(Math.pow(2, 53) - 1);  // true
```

32-bit (Signed) Integers

While integers can range up to roughly 9 quadrillion safely (53 bits), there are some numeric operations (like the bitwise operators) that are only defined for 32-bit numbers, so the "safe range" for numbers used in that way must be much smaller.

The range then is `Math.pow(-2,31)` (-2147483648, about -2.1 billion) up to `Math.pow(2,31)-1` (2147483647, about +2.1 billion).

9. The Non-value Values

For the `undefined` type, there is one and only one value: `undefined`. For the `null` type, there is one and only one value: `null`. So for both of them, the label is both its type and its value.

Both `undefined` and `null` are often taken to be interchangeable as either "empty" values or "non" values. Other developers prefer to distinguish between them with nuance. For example:

- `null` is an empty value
- `undefined` is a missing value

Or:

- `undefined` hasn't had a value yet
- `null` had a value and doesn't anymore

Regardless of how you choose to "define" and use these two values, `null` is a special keyword, not an identifier, and thus you cannot treat it as a variable to assign to (why would you!?). However, `undefined` is (unfortunately) an identifier. Uh oh.

```js
function foo() {
 undefined = 2; // really bad idea!
}

foo();
```

void Operator
The `void` operator "voids" out any value and returns `undefined`.

```js
console.log(void "test"); // undefined

var test = setTimeout(() => console.log("test"), 2000);
console.log(test); // TimeoutID positive integer in browser and object in Node.js

var void_test = void setTimeout(() => console.log(void "test"), 3000);
console.log(void_test); // undefined
```

If a value exists in some expression and you need it to return `undefined`, use the `void` operator.

10. NaN (Not a Number)

The Not a Number, Number 
A mathematic operation between both operands that are not numbers will produce a value `NaN` (not a number) which is still of type number.

`NaN` is a very special value and it's never equal to another `NaN` or itself. To check if something is `NaN`, we use the built-in helper function `isNaN()`.

For example:

```js
var a = 2 / "foo";  // NaN

typeof a === "number"; // true
```

In other words: "the type of not-a-number is 'number'!" Hooray for confusing names and semantics.

```js
var a = 2 / "foo";

Number.isNaN(a); // true
```

A few interesting things about `NaN`: `NaN` is never equal to `NaN`, the type of `NaN` is "number" (lol), and there's an important difference between `window.isNaN` and `Number.isNaN` (introduced in ES6).
 
```js
NaN === NaN //false
NaN == NaN //false

typeof NaN //"number"

/*
window.isNaN() will return true for actual NaN values *and* when 
the result is simply not a number.
*/
window.isNaN(2 / "foo") //true
window.isNaN("foo") //true

/*
Number.isNaN() was introduced with ES6. It means isNaN() will only
return true for actual NaN values.
*/
Number.isNaN(2 / "foo") //true
Number.isNaN("foo") //false
```

```js
var a = 1 / 0;  // Infinity
var b = -1 / 0; // -Infinity
```

```js 
1 / 0 //Infinity
-1 / -0 //Infinity
-1 / 0 //-Infinity
1 / -0 //-Infinity

Number.NEGATIVE_INFINITY //-Infinity
Number.POSITIVE_INFINITY //Infinity

1 / 0 === Number.POSITIVE_INFINITY //true
-1 / 0 === Number.NEGATIVE_INFINITY //true
```

11. `+0` and `-0` both exist in JS

Also did not know that JS has negative and positive zero values. Although, if you try to stringify a negative or positive zero, it will always be represented as `"0"`. Equality and comparison operators also don't work as expected with positive and negative zeros. But ES6's `Object.is()` can be used to accurately check equality for positive and negative zeros.

```js
0 / 3 //0
0 / -3 //-0
(0 / -3).toString() //"0"

0 === -0 //true
0 == -0 //true
0 > -0 //false
-0 < 0 //false

//ES6 Object.is() should only be used for special equality cases 
Object.is(0, -0) //false
```

```js
var a = 0 / -3;
a;     // -0

a.toString();   // "0"
String(a);   // "0"
JSON.stringify(a);  // "0"

+"-0";     // -0
Number("-0");   // -0
JSON.parse("-0");  // -0 inconsistent with JSON.stringify()

var a = 0;
var b = 0 / -3;

a == b;   // true
-0 == 0;  // true

a === b;  // true
-0 === 0; // true

0 > -0;   // false
a > b;    // false
```

Special Equality
The `Object.is()` method determines whether two values are the same value (ES6+).

```js
Object.is(NaN, NaN)  // true
Object.is(NaN, 1 / "a")  // true
Object.is(-3 * 0, -0)    // true
Object.is(-0, 0)    // false
```

Value vs. Reference
Simple scalar primitives (strings, numbers, ...) are assigned by value-copy but compound values (objects, arrays) are assigned by reference-copy. References are not like pointers in other languages. They are references to the value itself and not the variable.

```js
// value-copy
var a = 2;
var b = a;  // `b` is always a copy of the value in `a`
b++;
a; // 2
b; // 3

// reference-copy
var c = [1,2,3];
var d = c;  // `d` is a reference to the shared `[1,2,3]` value
d.push(4);
c; // [1,2,3,4]
d; // [1,2,3,4]
```

To empty an existing array without creating a new array we can use `Array.length = 0`. This is useful if an array is passed into function as a parameter.

```js
function foo(x) {
  x.push(4);
  x; // [1,2,3,4]

  // later
  x.length = 0; // empty existing array in-place
  /* 
  without the upper line the a; would be [1,2,3,4,5,6,7]
  if the upper line would be x = []; the a; would be [1,2,3,4] 
  because we made a reference to a newly created array and we didn't touch a again
  with the second push(4,5,6,7)
  */
  x.push(4,5,6,7);
  x; // [4,5,6,7]
}

var a = [1,2,3];

foo(a);

a; // [4,5,6,7]
```

Chapter 3: Natives

Natives
JavaScript has several native, or built-in, functions. These include:
- `String()`
- `Number()`  
- `Boolean()` 
- `Array()`
- `Object()`
- `Function()`
- `RegExp()`  
- `Date()`
- `Error()`
- `Symbol()`

It is true that each of these natives can be used as a native constructor. But what's being constructed may be different than you think.

The crux of this chapter on natives is this: You should only use the constructor form for value creation, i.e. `var x = new String("hello")`, for Dates and Errors (because there is no literal form). Everything else should be created literally, i.e. `var x = "hello"`.

Internal `[[Class]]`
    
Values that are `typeof "object"` (such as an array) are additionally tagged with an internal `[[Class]]` property (think of this more as an internal classification rather than related to classes from traditional class-oriented coding). This property cannot be accessed directly, but can generally be revealed indirectly by borrowing the default `Object.prototype.toString(..)` method called against the value. For example:

```js
Object.prototype.toString.call( [1,2,3] );   // "[object Array]"

Object.prototype.toString.call( /regex-literal/i ); // "[object RegExp]"
Object.prototype.toString.call( [1,2,3] );   // "[object Array]"
```

Object Wrappers or 'Boxing'

The constructor form results in an object wrapper around the primitive value. See `x` in the example below.

```js
var x = new String("hello")
var y = "hello"

console.log(x)
// String {[[PrimitiveValue]]: "hello"}

console.log(y)
// hello

typeof x
// "object"

typeof y
// "string"

x instanceof String
// true

y instanceof String
// false
```

Primitive values don't have properties or methods, so to access behaviour appropriate for the type, such as `.toString()` or `.length`, you need an object wrapper around the value. But JS automatically does this for us. It's even performance optimised.
In general, there's basically no reason to use the object form directly. It's better to just let the boxing happen implicitly where necessary. In other words, never do things like `new String("abc")`, `new Number(42)`, etc -- always prefer using the literal primitive values `"abc"` and `42`.

Boxing Wrappers
JavaScript automatically boxes (object wraps) primitive values with properties and methods of their object subtype.

```js
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"

Object.prototype.toString.call(a); // "[object String]"
```

Object Wrapper Gotchas
There are some gotchas with using the object wrappers directly that you should be aware of if you do choose to ever use them.

For example, consider Boolean wrapped values:

```js
var a = new Boolean(false);

if (!a) {
  console.log("Oops"); // never runs
}
```

The problem is that you've created an object wrapper around the `false` value, but objects themselves are "truthy" (see Chapter 4), so using the object behaves oppositely to using the underlying `false` value itself, which is quite contrary to normal expectation.  
If you want to manually box a primitive value, you can use the `Object(..)` function (no `new` keyword):

```js
var a = "abc";
var b = new String(a);
var c = Object(a);

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call(b); // "[object String]"
Object.prototype.toString.call(c); // "[object String]"
```

Unboxing
If we wanna access the primitive value under the object wrapper we can use `valueOf()` method.

```js
var a = new String("abc");
var b = new Number(42);
var c = new Boolean(true);

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

Unboxing can also happen implicitly, when using an object wrapper value in a way that requires the primitive value. This process (coercion) will be covered in more detail in Chapter 4, but briefly:

```js
var a = new String("abc");
var b = a + ""; // `b` has the unboxed primitive value "abc"
```

Natives as Constructors
For array, object, function, and regular-expression values, it's almost universally preferred that you use the literal form for creating the values.

There is no literal form for `Date(..)` and `Error(..)` native constructors.

```js
Date(); // "Wed Feb 20 2019 15:10:04 GMT+0100 (Central European Standard Time)"
new Date(); // Date object with current date and its methods
Date.now(); // 1550671999731 -> number of milliseconds elapsed since January 1, 1970 00:00:00 UTC

throw new Error("error message"); // useful for determining the position of the error in the code and call-stack.
```

Chapter 4: Coercion

Converting Values

```js
var a = 42;

var b = a + ""; // implicit coercion

var c = String(a); // explicit coercion
```

ToString

Very large and very small numbers are represented in exponential form.

```js
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;
a.toString(); // "1.07e21"

String(null) // "null"

String(undefined) // "undefined"

String(true) // "true"

String(false) // "false"

String(1) // "1"

String(10000000000 * 900000000000) // "9e+21"

String({}) // "[object Object]"

var x = {toString: () => '20'}
String(x) // "20"

String([1,2,3]) // "1,2,3"
```

JSON Stringification

JSON-safe value can be stringified using `JSON.stringify()`. JSON-safe is any value that doesn't include `undefined`, functions, symbols and objects. First three values are stringified as `null` but for the object the lookup occurs for the `toJSON()` function which should return a JSON-safe object (not string) which is then stringified by the `JSON.stringify()` function. Objects that have cyclic reference(s) are not JSON-safe and produce an error at stringification.

```js
var o = {};

var a = {
  b: 42,
  c: o,
  d: function(){}
};

o.e = a;

a.toJSON = function() {
  return { b: this.b };
};

JSON.stringify(a); // "{"b":42}"

// undefined, functions and symbols are excluded from objects

y = {a: 'hello', b: true, c: 123, d: undefined, e: () => {}, f: Symbol()}
JSON.stringify(y) // "{"a":"hello","b":true,"c":123}" 

// in an array, bad values are replaced with null 

y = [1, 'abc', undefined, () => {}, Symbol()]
JSON.stringify(y) // "[1,"abc",null,null,null]"

// a toJSON function can be added to the object to specify which properties you want to stringify 

y = {a: 1, b: '2', c: () => {}}
y.toJSON = function () {return {a: this.a}}
JSON.stringify(y) // "{"a":1}"

// a second argument can be used to specify which properties to include in the stringification  

JSON.stringify(y, ['b']) // "{}"

// a third argument can be used to apply indentation 

JSON.stringify(y, ['a'], 2)
"{
  "a": 1
}"
```

ToNumber

- `true` becomes `1`
- `false` becomes `0`  
- `undefined` becomes `NaN`
- `null` becomes `0`

```js
Number(null) // 0
Number(undefined) // NaN
Number(true) // 1
Number(false) // 0
Number(22) // 22
Number("hello") // NaN
Number("0") // 0
```

Objects and Arrays will first be converted to their primitive values and the resulting value is coerced to a number. 
If a `valueOf()` method is available and returns a primitive value, that is used for the coercion to number. If not, `toString()` will be used. If neither operation can provide a primitive value, a `TypeError` is thrown.

```js
x = {}
x.valueOf = () => 22
Number(x) // 22

y = []
y.toString = () => '22'
Number(y) // 22

z = {}
z.valueOf() // {} (not a primitive)
z.toString() // "[object Object]" (a string primitive which cannot be coerced to a number)
Number(z) // NaN
```

ToBoolean

"falsy" list:

- `undefined` 
- `null`
- `false`
- `+0`, `-0`, and `NaN`
- `""`

These values coerce to `false`. Anything not on the list coerces to `true`.

```js
var a = "0";
var b = "";

Boolean(a); // true
Boolean(b); // false

// falsy values
Boolean(undefined) // false
Boolean(null) // false 
Boolean(NaN) // false
Boolean(-0) // false
Boolean(+0) // false
Boolean("") // false

// everything else is truthy
Boolean(1) // true
Boolean(-1) // true
Boolean("a") // true
Boolean({}) // true
Boolean(() => {}) // true
Boolean([]) // true

// even object wrappers (created by the use of the word new) around falsy values are truthy 
Boolean(new Boolean(false)) // true
Boolean(new Boolean(0)) // true
Boolean(new Boolean("")) // true
```

Notice the last three examples in the above gist. The use of the keyword `new` means that `new Boolean(false)` results in an object wrapper around `false`, which we know is a falsy value. But `new Boolean(false)` will return `true` when coerced to a Boolean because objects are truthy. 

But the book explains that sometimes objects can be falsy. Ie, when coerced to a boolean, some objects will be `false`. These objects are not actually part of JavaScript. One example is `document.all`.

```js
document.all // HTMLAllCollection(224) [html, head, meta, …]
Boolean(document.all) // false

// we can also use double negation to coerce to Boolean
!!a; // true
!!b; // false
```

Explicitly: Parsing Numeric Strings

Explicitly coercing between String and Number
The most explicit way to coerce between a string and a number is to use `String()` and `Number()`. `String()` uses the rules of the ToString operation explained above, and `Number()` uses the rules of the ToNumber operation explained above.

Date to Number
This is something I always end up Googling—how to convert a date to milliseconds since epoch. The book explains how to do it with coercion, but recommends the non-coercion approach as it's even more explicit:

```js
// use for getting a specific (non-now) time
x = new Date().getTime() // 1518116090213
// use for getting the current time
y = Date.now() // 1518115963826
```

Parsing
Coercing a string to a number with `Number("12")` can also be achieved with `parseInt()`. Although there are some differences to how `Number()` and `parseInt()` work.

`Number()` results in `NaN` when given a non-numeric value, whereas `parseInt()` will return part of a string as a number. It reads the string from left to right and stops when it reaches a non-numeric value.

```js
var a = "42";
var b = "42px";

Number(a);  // 42
parseInt(a); // 42

Number(b);  // NaN
parseInt(b); // 42
```

The book suggests never to use `parseInt()` without a string and gives several examples of why.

Explicitly coercing to Boolean
`Boolean()` is an explicit way of forcing the ToBoolean operation explained above.
Probably the common explicit way of coercing to boolean is the double bang: `!!`.

Implicit Coercion

Implicitly coercing between String and Number
You can implicitly coerce from number to string with the `+` operator. Eg `1 + "" // "1"`.

Whether the `+` operator chooses to perform numeric addition or string concatenation is outlined in Section 11.6.1 of the ECMAScript specification. First, it performs the `toPrimitive` operation on the left and right hand side values. (Note that `toPrimitive` invokes `valueOf` on the value). If either of the values is then a string, it concatenates them.

There is a slight difference between implicitly coercing a number to string with `+` versus explicitly doing it with `String(…)`.
The `+` method invokes `valueOf()` while the explicit method invokes `toString()` directly. So when using an object, you may get different string values:

```js
a = { valueOf: () => 22, toString: () => 44 }
String(a) // "44"a + "" // "22"
```

You can implicitly coerce from string to number with the `-0`, `*1` or `/1`.

```js
"2" - 0 // 2"2" / 1 // 2"2" * 1 // 2
```

If either operand to `+` is a string (or becomes one), the operation will be string concatenation. Otherwise, it's always numeric addition. If an object (including array) is used for either operand, it first calls the `ToPrimitive` abstract operation on the value, which then calls the `[[DefaultValue]]` algorithm with a context hint of number.

```js
[1,2] + [3,4] // "1,23,4" = "1,2 + "3,4"

// [1,2].toString() + [3,4].toString();

[] + {} // "[object Object]" = "" + "[object Object]"

// [].toString() + new Object().toString();

var a = 42;
var b = a + "";

b; // "42"
```

For coercing from string to number we can use the `-`, `*`, `/` operators. These operators are only defined for numeric operations.

```js
var a = "3.14";
var b = a - 0;
b; // 3.14

var c = [3];
var d = [1];
c - d; // 2
```

Implicitly: `* --> Boolean`

The book lists 5 occasions where a value will be implicitly coerced to a boolean, if it isn't already one.

1. `if(..)` statement
2. `for(..; ..; ..)` (second clause) in loop 
3. `while(..)` and `do..while(..)` loops
4. `? :` (first clause) in ternary expressions
5. `||` ("logical or") and `&&` ("logical and") operators in a test expression

Operators `||` and `&&`

The super interesting thing I learned in this section — which seems obvious in hindsight — is that `||` and `&&` don't actually result in a boolean.
For example, `x = 22 && "tay"` will not result in `true`, as you might expect because both values are truthy. `x` will instead result in `"tay"`. So `&&` and `||` will always result in one of the values in the expression, which may then be coerced to a boolean.
For example, `if (22 && "tay") {…}` would implicitly coerce `"tay"` to `true`. The book goes into great detail about how `&&` and `||` will either result in the value on the left or right hand side of the operator. 

Symbol Coercion
Explicit coercion from symbol to string is allowed but implicit throws an error.

```js
var s1 = Symbol("cool");
String(s1); // "Symbol(cool)"

var s2 = Symbol("not cool");
s2 + ""; // TypeError: can't convert symbol to string
```

Symbol values cannot coerce to number but they can explicitly and implicitly coerce to boolean (always `true`).

Loose Equals vs. Strict Equals

It's commonly thought that `===` cares about types whereas `==` doesn't. The book explains that more correctly, `==` allows coercion in the equality comparison, while `===` disallows coercion. 
The book says this does not mean `===` does more work, and says performance should not be a factor when deciding whether to use `==` or `===`. Interestingly, it says the exact same algorithm is applied when comparing two JavaScript objects, regardless of whether `==` or `===` is used.
If you want coercion, use `==` loose equality, but if you don't want coercion, use `===` strict equality.

```js
// strings to numbers
var a = 42;
var b = "42";

a == b;   // true  (with coercion)  42 = 42
a === b; // false (without coercion) 42 = "42"

// anything to boolean
var a = "42";
var b = true;

a == b; // false 42 == 1 (both get coerced to number)

// null to undefined
var a = null;
var b;

a == b;   // true
a == null; // true
b == null; // true

a == false; // false
b == false; // false
a == "";  // false
b == "";  // false
a == 0;   // false
b == 0;   // false

// objects to non-objects
var a = 42;
var b = [42];

a == b; // true  42 == "42" with coercion 42 == 42
```

The most interesting points are 6 and 7. They describe what happens when a boolean is on either side of the `==`. For example, you might expect `"22" == true` to return `true`, because `"22"` is truthy. But it returns `false`. According to spec (see above), the value `true` is first coerced to a number with the `ToNumber` operation. This would result in `1`. Then we are comparing a string and number, `"22" == 1`. `"22"` then will get also coerced to a number. So the final comparison is `22 == 1`, which of course is `false`.

```js
"22" == true // false"22" == 1 // false22 == 1 // false
```

The book recommends to never use `== true` or `== false` in your code.
`var a = 22`. Instead of `if (a == true) {…}`, which would coerce the boolean to a number, use `if (a) {…}` (for an implicit boolean coercion) or `if (Boolean(a)) {…}` (for an explicit boolean coercion).
Also noteworthy is that `null` and `undefined` are equivalent to each other with `==`. `null == undefined` returns `true`. `if (a ==null) {… }` and `if (a === null || a === undefined ) {…}` are doing the same check.
When an object is on either side of the `==` comparison, the object will first be run through the `ToPrimitive` function.

```js
var a = "12"var b = new String("12")
a // "12"b // String {"12"}
a == b // true (a is compared with b's toPrimitive value)
// b's primitive value is obtained by calling toString()b.toString() // "12"
```

Edge cases
As seen in the above gist and detailed in the spec, an object on either side of a `==`, where the value on the other side is a string or number, will be coerced with `ToPrimitive`. This coercion ultimately results in `Number.prototype.toString()` or `Number.prototype.valueOf()` being called. The book outlines some edge cases where `toString` or `valueOf` are modified, for example to always return the value `3`, in which case `new Number(2) == 3` would result in `true`.

7 gotcha coercions
The book says apart from cases above where `valueOf` or `toString` may have been intentionally modified, there are seven "gotcha" `==` comparisons to be aware of.
Here are the gotchas and my explanations of them:

```js
false == "0" // true// false gets coerced to 0// "0" gets coerced to 0// 0 == 0 is true 
false == 0 // true// false gets coerced to 0// 0 == 0 is true
false == "" // true// false gets coerced to 0// "" gets coerced to a number, 0// 0 == 0 is true
false == [] // true// false gets coerced to 0// [] is an object so ToPrimitive would be called on it// valueOf() would be consulted to find a primitive// [].valueOf() would result in [], not a primitive value// so [].toString() would be called, resulting in ""// "" would then get coerced to a number, 0// 0 == 0 is true
"" == 0 // true// "" gets coerced to a number, 0// 0 == 0 is true
"" == [] // true// [] is an object so ToPrimitive would be called on it// valueOf() would be consulted to find a primitive// [].valueOf() would result in [], not a primitive value// so [].toString() would be called, resulting in ""// "" == "" is true
0 == [] // true// [] is an object so ToPrimitive would be called on it// valueOf() would be consulted to find a primitive// [].valueOf() would result in [], not a primitive value// so [].toString() would be called, resulting in ""// "" would then get coerced to a number, 0// 0 == 0 is true
```

Conclusions on implicit coercion
The book gives some rules to help avoid issues with `==` comparisons. It says:

1. If either side of the comparison can have `true` or `false` values, don't ever, EVER use `==`.
2. If either side of the comparison can have `[]`, `""`, or `0` values, seriously consider not using `==`.

In these scenarios, it's almost certainly better to use `===` instead of `==`, to avoid unwanted coercion. Follow those two simple rules and pretty much all the coercion gotchas that could reasonably hurt you will effectively be avoided.
It says the question of whether to use `==` versus `===` should be framed around whether you want to allow coercion for the comparison or not.

`>` and `<` comparisons
The algorithm is only defined for `<`, so `a > b` is reversed and evaluated as `b < a`.

As outlined in the Abstract Relational Comparison Algorithm section of the spec, `ToPrimitive` (with a hint of number) is called on both values. 
If both primitive values are not strings, then both are coerced to number and compared numerically.
If both primitive values are strings, then natural alphabetic comparison on the characters is performed. Eg `["22"] < ["23"]` would return `true`, because the primitive values are `"22"` and `"23"`.
`{a: 42} < {a: 43}` and `{a: 42} > {a: 43}` both return `false`, because the primitive value is `"[object Object]"` for both sides of the comparison.

Side note: You might think `{a: 42} == {a: 43}` would return `true`. It actually returns `false`, because when comparing two objects with `==` or `===`, they will only be equal if they are both references to the exact same object. So no coercion occurs.

Interestingly, `{a: 42} <= {a: 43}` and `{a: 42} >= {a: 43}` both return `true`. For `{a: 42} <= {a: 43}` the spec will "let r be the result of performing abstract relational comparison rval < lval" (which in this case would be `{a: 43} < {a: 42` and returns `false`). The spec says "if r is true or undefined, return false. Otherwise, return true." What about `{a: 42} >= {a: 43}`? The spec will "let r be the result of performing abstract relational comparison lval < rval" (which in this case would be `{a: 42} < {a: 43}`, and returns `false`). The spec says "if r is true or undefined, return false. Otherwise, return true."

The book says "less than or equal to" is more accurately phrased as "not greater than".

This is tripping me up a little. So in the case of `1 <= 2`, it would first evaluate `2 < 1`, which is `false`, which gets negated to `true`. In the case of `2 <= 2` it would evaluate `2 < 2`, which is `false`, which gets negated to `true`. What about `1 >= 2`. It would evaluate `1 < 2`, which is `true`, which gets negated to `false`.

OK, makes sense. So with `<=` JS will evaluate by performing `rightHandSideValue < leftHandsideValue`, and then negating the result. And with `>=` JS will evaluate by performing `leftHandsideValue < rightHandSideValue`, and then negating the result.

Abstract Relational Comparison
The algorithm first calls `ToPrimitive` coercion on both values, and if the return result of either call is not a string, then both values are coerced to number values using the `ToNumber` operation rules, and compared numerically.

```js
var a = [42];
var b = ["43"];

a < b;  // true  42 < 43
b < a;  // false 43 < 42
```

If `ToPrimitive` call coerces to two string values they are compared lexicographically.

```js
var a = ["42"];
var b = ["043"];

a < b;  // false
```

Grammar
Statements & Expressions

Statement completion values
Something I've never known the answer to is why typing `var a = 2 * 2` into the developer console will result in `undefined` being printed. 
The book explains how `var a = 2` is a statement containing an expression. In this case it is a declaration statement, because it declares a variable and optionally assigns a value (the result of the `2 * 2` expression) to it.
The book explains how every statement has a completion value, which is what gets printed in the console when the statement is executed. For `var` statements, the completion value is `undefined`.
It's possibly the most boring explanation for something that seems so unusual!

JSON-P
I don't know much about JSON-P but I found this interesting. A valid JSON value like `{"a": 42}` would throw an error in JavaScript. This is because of a feature in JS called label statements. In the statement `{a: 42}`, without any `var x =` in front of it, `a` is a label for the statement `42`. But a labelled statement cannot contain quotation marks around the label. So in the `{"a": 42}` example, the code is being read a labelled statement, not as JSON, and is throwing an error due to the quotation marks.

This is where JSON-P comes into the picture. JSON-P passes the valid JSON syntax `{"a": 42}` to a function, where instead of being read as a label statement, it gets read as an object literal value.

```js
{"a": 42} // SyntaxError: Unexpected token ':'. Parse error.
function foo (data) {
  console.log(data.a)
}
foo({"a":42}) // 42
```

Automatic semicolons
So I've heard of the debate about whether or not to use semicolons in JS. But until reading the section on Automatic Semicolon Insertion (ASI) I never understood what happens if you omit them. JS puts them back in for you!

I really enjoyed the author's perspective on this:
A strict reading of the spec implies that ASI is an "error correction" routine. What kind of error, you may ask? Specifically, a parser error…
In my view, the only way a parser error occurs is if it's given an incorrect/errored program to parse. So, while ASI is strictly correcting parser errors, the only way it can get such errors is if there were first program authoring errors — omitting semicolons where the grammar rules require them…
When I hear someone claim that they want to omit "optional semicolons," my brain translates that claim to "I want to write the most parser-broken program I can that will still work."…
My take: use semicolons wherever you know they are "required," and limit your assumptions about ASI to a minimum.

Statements are sentences, expressions are phrases, and operators are conjunctions/punctuation.

```js
var a = 3 * 6; // declaration statement but without var its assignment expression
var b = a; // -||-
b; // expression statement
// *,= are operators here
```

Expression Side Effects

```js
var a = 42;
var b = a++; // postfix increment which happens after the value is returned from the expression

a; // 43
b; // 42
```

Operator Precedence

```js
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

d;  // 42
((a && b) || c) ? ( (c || b) ? a : (c && b) ) : a  // precedence: && over || over ( ? : )
("foo" || c) ? ( (c || b) ? a : (false) ) : a  // evaluated && operator
"foo" ? ("foo" ? a : false ) : a   // evaluated || operator
"foo" ? 42 : 42      // evaluated inner ternary operator
42        // evaluated outer ternary operator
```

`try..finally`
The code in the `finally` clause always runs and also after the `try` and `catch` clause finish. If we have a `return` or `throw` statement in the `finally` clause that will also override as the primary completion of that function. The previous `return` value for `try` or `catch` clause will be abandoned.

```js
function foo() {
  try {
    console.log("console try");
    throw "throw try";
  }
  catch (e) {
    console.log("console catch");
    throw "catch";
  }
  finally {
    console.log("console finally");
    throw "finally"
  }

  console.log("never runs");
}

console.log(foo());
/*

console try
console catch
console finally
Error: finally

*/
```

`switch`
The matching that occurs in the `case` expression is identical to the `===` algorithm. We can force the coercive equality (`==`) with the next example. The `default` clause doesn't need to be at the end.

```js
var a = "42";

switch (true) {
  case a == 10:
    console.log("10 or '10'");
    break;
  case a == 42:
    console.log("42 or '42'");
    break;
  default:
    // never gets here
}
// 42 or '42'
```
