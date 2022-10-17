# You Don't Know JS: Types & Grammar

<details>
<summary>Built-In Types</summary>

JavaScript defines seven built-in types:

- null
- undefined
- boolean
- number
- string
- object
- symbol -- added in ES6!

Note: All of these types except object are called "primitives".

```
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true

// added in ES6!
typeof Symbol()      === "symbol";    // true

typeof null === "object"; // true
```

If you want to test for a null value using its type, you need a compound condition:

```
var a = null;

(!a && typeof a === "object"); // true
```

</details>

<details>
<summary>undefined vs "undeclared"</summary>
  
Many developers will assume "undefined" and "undeclared" are roughly the same thing, but in JavaScript, they're quite different. undefined is a value that a declared variable can hold. "Undeclared" means a variable has never been declared.
  
```
var a;

a; // undefined
b; // ReferenceError: b is not defined
```

The typeof operator returns "undefined" even for "undeclared" (or "not defined") variables. Notice that there was no error thrown when we executed typeof b, even though b is an undeclared variable. This is a special safety guard in the behavior of typeof:
  
```
var a;

typeof a; // "undefined"

typeof b; // "undefined"
```

JavaScript unfortunately kind of conflates these two terms, not only in its error messages ("ReferenceError: a is not defined") but also in the return values of typeof, which is "undefined" for both cases.

However, the safety guard (preventing an error) on typeof when used against an undeclared variable can be helpful in certain cases.
</details>

<details>
<summary>Arrays</summary>
  
arrays are numerically indexed (as you'd expect), but the tricky thing is that they also are objects that can have string keys/properties added to them (but which don't count toward the length of the array):
  
```
var a = [ ];

a[0] = 1;
a["foobar"] = 2;

a.length;		// 1
a["foobar"];	// 2
a.foobar;		// 2
```
  
However, a gotcha to be aware of is that if a string value intended as a key can be coerced to a standard base-10 number, then it is assumed that you wanted to use it as a number index rather than as a string key!
  
```
var a = [ ];

a["13"] = 42;

a.length; // 14
```
</details>


<details>
<summary>array vs string</summary>
JavaScript strings are immutable, while arrays are quite mutable:
  
```
a[1] = "O";
b[1] = "O";

a; // "foo"
b; // ["f","O","o"]
```

A further consequence of immutable strings is that none of the string methods that alter its contents can modify in-place, but rather must create and return new strings. By contrast, many of the methods that change array contents actually do modify in-place:
  
```
c = a.toUpperCase();
a === c;	// false
a;			// "foo"
c;			// "FOO"

b.push( "!" );
b;			// ["f","O","o","!"]
```
  

</details>

<details>
<summary>.toFixed() vs .toPrecision()</summary>
  
```
var a = 42.59;

a.toFixed( 0 ); // "43"
a.toFixed( 1 ); // "42.6"
a.toFixed( 2 ); // "42.59"
a.toFixed( 3 ); // "42.590"
a.toFixed( 4 ); // "42.5900"
```
  
```
var a = 42.59;

a.toPrecision( 1 ); // "4e+1"
a.toPrecision( 2 ); // "43"
a.toPrecision( 3 ); // "42.6"
a.toPrecision( 4 ); // "42.59"
a.toPrecision( 5 ); // "42.590"
a.toPrecision( 6 ); // "42.5900"
```

But you have to be careful with the . operator. Since . is a valid numeric character, it will first be interpreted as part of the number literal, if possible, instead of being interpreted as a property accessor:
  
```
// invalid syntax:
42.toFixed( 3 );	// SyntaxError

// these are all valid:
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	// "42.000"
```

</details>

<details>
<summary>Use case for the void operator</summary>

But the void operator can be useful in a few other circumstances, if you need to ensure that an expression has no result value (even if it has side effects).

For example:
  
```
  function doSomething() {
	// note: `APP.ready` is provided by our application
	if (!APP.ready) {
		// try again later
		return void setTimeout( doSomething, 100 );
	}

	var result;

	// do some other stuff
	return result;
}

// were we able to do it right away?
if (doSomething()) {
	// handle next tasks right away
}
  ```
  
  Here, the setTimeout(..) function returns a numeric value (the unique identifier of the timer interval, if you wanted to cancel it), but we want to void that out so that the return value of our function doesn't give a false-positive with the if statement.

Many devs prefer to just do these actions separately, which works the same but doesn't use the void operator:
  
  ```
  if (!APP.ready) {
	// try again later
	setTimeout( doSomething, 100 );
	return;
}
  ```
  
  In general, if there's ever a place where a value exists (from some expression) and you'd find it useful for the value to be undefined instead, use the void operator. That probably won't be terribly common in your programs, but in the rare cases you do need it, it can be quite helpful.
</details>

<details>
<summary>NaN</summary>
  
  In other words: "the type of not-a-number is 'number'!"
  ```
  var a = 2 / "foo";		// NaN

typeof a === "number";	// true
  ```
  
  The isNaN(..) utility has a fatal flaw. It appears it tried to take the meaning of NaN ("Not a Number") too literally -- that its job is basically: "test if the thing passed in is either not a number or is a number." But that's not quite accurate.
  
  ```
  var a = 2 / "foo";
var b = "foo";

a; // NaN
b; // "foo"

window.isNaN( a ); // true
window.isNaN( b ); // true -- ouch!
  ```
  
  Clearly, "foo" is literally not a number, but it's definitely not the NaN value either! This bug has been in JS since the very beginning (over 19 years of ouch).
  
  As of ES6, finally a replacement utility has been provided: Number.isNaN(..).
  
</details>


<details>
<summary>0 vs -0</summary>

If you want to distinguish a -0 from a 0 in your code, you can't just rely on what the developer console outputs, so you're going to have to be a bit more clever:

	```
	function isNegZero(n) {
	n = Number( n );
	return (n === 0) && (1 / n === -Infinity);
}

isNegZero( -0 );		// true
isNegZero( 0 / -3 );	// true
isNegZero( 0 );			// false
	```
	
	Now, why do we need a negative zero, besides academic trivia?

There are certain applications where developers use the magnitude of a value to represent one piece of information (like speed of movement per animation frame) and the sign of that number to represent another piece of information (like the direction of that movement).

In those applications, as one example, if a variable arrives at zero and it loses its sign, then you would lose the information of what direction it was moving in before it arrived at zero. Preserving the sign of the zero prevents potentially unwanted information loss.
</details>

<details>
<summary>Special Equality</summary>
	
	As we saw above, the NaN value and the -0 value have special behavior when it comes to equality comparison. NaN is never equal to itself, so you have to use ES6's Number.isNaN(..) (or a polyfill). Similarly, -0 lies and pretends that it's equal (even === strict equal -- see Chapter 4) to regular positive 0, so you have to use the somewhat hackish isNegZero(..) utility we suggested above.

As of ES6, there's a new utility that can be used to test two values for absolute equality, without any of these exceptions. It's called Object.is(..):
	
	```
	var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN );	// true
Object.is( b, -0 );		// true

Object.is( b, 0 );		// false
	```

</details>

<details>
<summary>Value vs Reference</summary>
	
	```
	function foo(x) {
	x.push( 4 );
	x; // [1,2,3,4]

	// later
	x = [4,5,6];
	x.push( 7 );
	x; // [4,5,6,7]
}

var a = [1,2,3];

foo( a );

a; // [1,2,3,4]  not  [4,5,6,7]
	```
	
	When we pass in the argument a, it assigns a copy of the a reference to x. x and a are separate references pointing at the same [1,2,3] value. Now, inside the function, we can use that reference to mutate the value itself (push(4)). But when we make the assignment x = [4,5,6], this is in no way affecting where the initial reference a is pointing -- still points at the (now modified) [1,2,3,4] value.
</details>

<details>
<summary></summary>
</details>
