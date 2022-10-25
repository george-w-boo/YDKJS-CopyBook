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

isNegZero( -0 ); // true
isNegZero( 0 / -3 );  // true
isNegZero( 0 );	// false

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

Object.is( a, NaN );  // true
Object.is( b, -0 );  // true

Object.is( b, 0 );  // false

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
<summary>String()</summary>

```

var a = new String( "abc" );

typeof a; // "object" ... not "String"

a instanceof String; // true

Object.prototype.toString.call( a ); // "[object String]"

```

The point is, new String("abc") creates a string wrapper object around "abc", not just the primitive "abc" value itself.

</details>

<details>
<summary>Object Wrapper Gotchas</summary>

There are some gotchas with using the object wrappers directly that you should be aware of if you do choose to ever use them.

For example, consider Boolean wrapped values:

```

var a = new Boolean( false );

if (!a) {
	console.log( "Oops" ); // never runs
}

```

The problem is that you've created an object wrapper around the false value, but objects themselves are "truthy" (see Chapter 4), so using the object behaves oppositely to using the underlying false value itself, which is quite contrary to normal expectation.

If you want to manually box a primitive value, you can use the Object(..) function (no new keyword):

```

var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"

```

Again, using the boxed object wrapper directly (like b and c above) is usually discouraged, but there may be some rare occasions you'll run into where they may be useful.

</details>

<details>
<summary>Array as a native constructor</summary>

So, if you wanted to actually create an array of actual undefined values (not just "empty slots"), how could you do it (besides manually)?

```

var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]

```

Confused? Yeah. Here's roughly how it works.

apply(..) is a utility available to all functions, which calls the function it's used with but in a special way.

The first argument is a this object binding (covered in the this & Object Prototypes title of this series), which we don't care about here, so we set it to null. The second argument is supposed to be an array (or something like an array -- aka an "array-like object"). The contents of this "array" are "spread" out as arguments to the function in question.

So, Array.apply(..) is calling the Array(..) function and spreading out the values (of the { length: 3 } object value) as its arguments.

Inside of apply(..), we can envision there's another for loop (kinda like join(..) from above) that goes from 0 up to, but not including, length (3 in our case).

For each index, it retrieves that key from the object. So if the array-object parameter was named arr internally inside of the apply(..) function, the property access would effectively be arr[0], arr[1], and arr[2]. Of course, none of those properties exist on the { length: 3 } object value, so all three of those property accesses would return the value undefined.

In other words, it ends up calling Array(..) basically like this: Array(undefined,undefined,undefined), which is how we end up with an array filled with undefined values, and not just those (crazy) empty slots.

</details>

<details>
<summary>Converting Values</summary>

Converting a value from one type to another is often called "type casting," when done explicitly, and "coercion" when done implicitly (forced by the rules of how a value is used).

Another way these terms are often distinguished is as follows: "type casting" (or "type conversion") occur in statically typed languages at compile time, while "type coercion" is a runtime conversion for dynamically typed languages.

However, in JavaScript, most people refer to all these types of conversions as coercion, so the way I prefer to distinguish is to say "implicit coercion" vs. "explicit coercion."

```

var a = 42;

var b = a + "";	// implicit coercion

var c = String( a );  // explicit coercion

```

</details>

<details>
<summary>.toJSON vs JSON.stringify</summary>

It's a very common misconception that toJSON() should return a JSON stringification representation. That's probably incorrect, unless you're wanting to actually stringify the string itself (usually not!). toJSON() should return the actual regular value (of whatever type) that's appropriate, and JSON.stringify(..) itself will handle the stringification.

In other words, toJSON() should be interpreted as "to a JSON-safe value suitable for stringification," not "to a JSON string" as many developers mistakenly assume.

Consider:

```

var a = {
	val: [1,2,3],

	// probably correct!
	toJSON: function(){
		return this.val.slice( 1 );
	}
};

var b = {
	val: [1,2,3],

	// probably incorrect!
	toJSON: function(){
		return "[" +
			this.val.slice( 1 ).join() +
		"]";
	}
};

JSON.stringify( a ); // "[2,3]"

JSON.stringify( b ); // ""[2,3]""

```

In the second call, we stringified the returned string rather than the array itself, which was probably not what we wanted to do.

Remember, JSON.stringify(..) is not directly a form of coercion. We covered it here, however, for two reasons that relate its behavior to ToString coercion:

string, number, boolean, and null values all stringify for JSON basically the same as how they coerce to string values via the rules of the ToString abstract operation.
If you pass an object value to JSON.stringify(..), and that object has a toJSON() method on it, toJSON() is automatically called to (sort of) "coerce" the value to be JSON-safe before stringification.

```

</details>

<details>
<summary>Second argument for JSON.stringify</summary>

An optional second argument can be passed to JSON.stringify(..) that is called replacer. This argument can either be an array or a function. It's used to customize the recursive serialization of an object by providing a filtering mechanism for which properties should and should not be included, in a similar way to how toJSON() can prepare a value for serialization.

```

var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"

```

Note: In the function replacer case, the key argument k is undefined for the first call (where the a object itself is being passed in). The if statement filters out the property named "c". Stringification is recursive, so the [1,2,3] array has each of its values (1, 2, and 3) passed as v to replacer, with indexes (0, 1, and 2) as k.

</details>

<details>
<summary>Optional third argument for JSON.stringify</summary>

A third optional argument can also be passed to JSON.stringify(..), called space, which is used as indentation for prettier human-friendly output. space can be a positive integer to indicate how many space characters should be used at each indentation level. Or, space can be a string, in which case up to the first ten characters of its value will be used for each indentation level.

```

var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"

</details>

<details>
<summary>Falsy objects</summary>

They are not these:

```

var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );

```

So, if "falsy objects" are not just objects wrapped around falsy values, what the heck are they?

The tricky part is that they can show up in your JS program, but they're not actually part of JavaScript itself.

A "falsy object" is a value that looks and acts like a normal object (properties, etc.), but when you coerce it to a boolean, it coerces to a false value.

Why!?

The most well-known case is document.all: an array-like (object) provided to your JS program by the DOM (not the JS engine itself), which exposes elements in your page to your JS program. It used to behave like a normal object--it would act truthy. But not anymore.

So... that's what we've got: crazy, nonstandard "falsy objects" added to JavaScript by the browsers. Yay!

</details>

<details>
<summary>Use case for ~</summary>

```
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- truthy!

if (~a.indexOf( "lo" )) {	// true
	// found it!
}

~a.indexOf( "ol" );			// 0    <-- falsy!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// not found!
}

```

~ takes the return value of indexOf(..) and transforms it: for the "failure" -1 we get the falsy 0, and every other value is truthy.

Note: The -(x+1) pseudo-algorithm for ~ would imply that ~-1 is -0, but actually it produces 0 because the underlying operation is actually bitwise, not mathematic.

</details>

<details>
<summary>~~ vs Math.floor(..)</summary>

 ~~ needs some caution/clarification. First, it only works reliably on 32-bit values. But more importantly, it doesn't work the same on negative numbers as Math.floor(..) does!

```

	Math.floor( -49.6 );	// -50
~~-49.6;  // -49

```

</details>

<details>
<summary>Implicitly: Strings <--> Numbers</summary>

```

var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"

```

According to ES5 spec section 11.6.1, the + algorithm (when an object value is an operand) will concatenate if either operand is either already a string, or if the following steps produce a string representation. So, when + receives an object (including array) for either operand, it first calls the ToPrimitive abstract operation (section 9.1) on the value, which then calls the [[DefaultValue]] algorithm (section 8.12.8) with a context hint of number.

If you're paying close attention, you'll notice that this operation is now identical to how the ToNumber abstract operation handles objects (see the "ToNumber"" section earlier). The valueOf() operation on the array will fail to produce a simple primitive, so it then falls to a toString() representation. The two arrays thus become "1,2" and "3,4", respectively. Now, + concatenates the two strings as you'd normally expect: "1,23,4".

____

Comparing this implicit coercion of a + "" to our earlier example of String(a) explicit coercion, there's one additional quirk to be aware of. Because of how the ToPrimitive abstract operation works, a + "" invokes valueOf() on the a value, whose return value is then finally converted to a string via the internal ToString abstract operation. But String(a) just invokes toString() directly.

Both approaches ultimately result in a string, but if you're using an object instead of a regular primitive number value, you may not necessarily get the same string value!

Consider:

```
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"

```

Generally, this sort of gotcha won't bite you unless you're really trying to create confusing data structures and operations, but you should be careful if you're defining both your own valueOf() and toString() methods for some object, as how you coerce the value could affect the outcome.

____

What about the other direction? How can we implicitly coerce from string to number?

```

var a = "3.14";
var b = a - 0;

b; // 3.14

```

The - operator is defined only for numeric subtraction, so a - 0 forces a's value to be coerced to a number. While far less common, a * 1 or a / 1 would accomplish the same result, as those operators are also only defined for numeric operations.

</details>

<details>
<summary>Operators || and &&</summary>

The value produced by a && or || operator is not necessarily of type Boolean. The value produced will always be the value of one of the two operand expressions:

```

var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
	
```

Both || and && operators perform a boolean test on the first operand (a or c). If the operand is not already boolean (as it's not, here), a normal ToBoolean coercion occurs, so that the test can be performed.

For the || operator, if the test is true, the || expression results in the value of the first operand (a or c). If the test is false, the || expression results in the value of the second operand (b).

Inversely, for the && operator, if the test is true, the && expression results in the value of the second operand (b). If the test is false, the && expression results in the value of the first operand (a or c).

The result of a || or && expression is always the underlying value of one of the operands, not the (possibly coerced) result of the test. In c && b, c is null, and thus falsy. But the && expression itself results in null (the value in c), not in the coerced false used in the test.

In fact, I would argue these operators shouldn't even be called "logical ___ operators", as that name is incomplete in describing what they do. If I were to give these operators a more accurate (if more clumsy) name, I'd call them "selector operators," or more completely, "operand selector operators."

An extremely common and helpful usage of this behavior, which there's a good chance you may have used before and not fully understood, is:

```

function foo(a,b) {
	a = a || "hello";
	b = b || "world";

	console.log( a + " " + b );
}

foo();					// "hello world"
foo( "yeah", "yeah!" );	// "yeah yeah!"

```

This || idiom is extremely common, and quite helpful, but you have to use it only in cases where all falsy values should be skipped. Otherwise, you'll need to be more explicit in your test, and probably use a ? : ternary instead.
</details>
	
<details>
<summary>Symbol Coercion</summary>

Explicit coercion of a symbol to a string is allowed, but implicit coercion of the same is disallowed and throws an error:

```

var s1 = Symbol( "cool" );
String( s1 );					// "Symbol(cool)"

var s2 = Symbol( "not cool" );
s2 + "";						// TypeError

```

symbol values cannot coerce to number at all (throws an error either way), but strangely they can both explicitly and implicitly coerce to boolean (always true).

</details>

<details>
<summary>Loose Equals vs. Strict Equals</summary>

"== allows coercion in the equality comparison and === disallows coercion."
	
It's a very little known fact that == and === behave identically in the case where two objects are being compared!

Let's again quote the spec, clauses 11.9.3.6-7 for loose ==:

If Type(x) is Boolean, return the result of the comparison ToNumber(x) == y.
If Type(y) is Boolean, return the result of the comparison x == ToNumber(y).
	
"42" == true is not performing a boolean test/coercion at all, no matter what your brain says. "42" is not being coerced to a boolean (true), but instead true is being coerced to a 1, and then "42" is being coerced to 42.

Another example of implicit coercion can be seen with == loose equality between null and undefined values. Yet again quoting the ES5 spec, clauses 11.9.3.2-3:

If x is null and y is undefined, return true.
If x is undefined and y is null, return true.

Which is why no need to do sth like this:
	
```
	
var a = doSomething();

if (a === undefined || a === null) {
	// ..
}

```

</details>

<details>
<summary></summary>
</details>
