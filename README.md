# Objects & Classes
My conspectus of the Objects & Classes of the YDKJSY book series
____

<details>
  <summary>Object property descriptor</summary>

  ```
  myObj = {
      favoriteNumber: 42,
      isDeveloper: true,
      firstName: "Kyle"
  };

  Object.getOwnPropertyDescriptor(myObj,"favoriteNumber");
  // {
  //     value: 42,
  //     enumerable: true,
  //     writable: true,
  //     configurable: true
  // }
  ```

The <b>enumerable</b> attribute controls whether the property will appear in various enumerations of object properties, such as `Object.keys(..)`, `Object.entries(..)`, `for..in` loops, and the copying that occurs with the ... object spread and `Object.assign(..)`. Most properties should be left enumerable, but you can mark certain special properties on an object as non-enumerable if they shouldn't be iterated/copied.

The <b>writable</b> attribute controls whether a value assignment (via =) is allowed. To make a property "read only", define it with `writable: false`. However, as long as the property is still configurable, `Object.defineProperty(..)` can still change the value by setting value differently.

The <b>configurable</b> attribute controls whether a property's descriptor can be re-defined/overwritten.

</details>

<details>
<summary>Avoid Setting Function-Object Properties</summary>

You should avoid assigning properties on function objects. If you're looking to store extra information associated with a function, use a separate `Map(..)` (or `WeakMap(..)`) with the function object as the key, and the extra information as the value.
extraInfo = new Map();

`extraInfo.set(help,"this is some important information");`

```
// later:
extraInfo.get(help);   // "this is some important information"
```

</details>


<details>
<summary>Configurint the whole object</summary>

In addition to defining behaviors for specific properties, certain behaviors are configurable across the whole object:
* extensible - `Object.preventExtensions(objToBeNotExtensible)` - no more properties can be defined/added
* sealed - `Object. seal(objToBeSealed)` - prevents new properties from being added to it and marks all existing properties as non-configurable
* frozen -  `Object.freeze(objectToBeFrozen)` - new properties cannot be added, existing properties cannot be removed, their enumerability, configurability, writability, or value cannot be changed, and the object's prototype cannot be re-assigned

</details>


<details>
<summary></summary>
</details>

