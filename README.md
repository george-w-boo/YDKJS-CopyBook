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
<summary></summary>
</details>
