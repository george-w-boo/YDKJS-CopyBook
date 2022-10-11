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
<summary></summary>
</details>
