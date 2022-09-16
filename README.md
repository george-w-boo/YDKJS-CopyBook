# Objects & Classes
My conspectus of the Objects & Classes of the YDKJSY book series
____

## Chapter 1: Object Foundations

<details>
<summary>Destructuring</summary>
  
```
myObj = {
    favoriteNumber: 42,
    isDeveloper: true,
    firstName: "Kyle"
};

const { favoriteNumber = 12 } = myObj;
const {
    isDeveloper: isDev,
    firstName: firstName,
    lastName: lname = "--missing--"
} = myObj;

favoriteNumber;   // 42
isDev;            // true
firstName;        // "Kyle"
lname;            // "--missing--"
```
</details>

<details>
<summary>for..in vs Object.hasOwnProperty() vs Object.hasOwn()</summary>
  
There is an important difference between how the in operator and the hasOwnProperty(..) method behave. The in operator will check not only the target object specified, but if not found there, it will also consult the object's [[Prototype]] chain (covered in the next chapter). By contrast, hasOwnProperty(..) only consults the target object.
  
ES2022 (almost official at time of writing) has already settled on a new feature, Object.hasOwn(..). It does essentially the same thing as hasOwnProperty(..), but it's invoked as a static helper external to the object value instead of via the object's [[Prototype]], making it safer and more consistent in usage

</details>

<details>
<summary>List all object's own keys including Symbols</summary>
  
But what if we wanted to get all the keys in an object (enumerable or not)? Object.getOwnPropertyNames(..) seems to do what we want, in that it's like Object.keys(..) but also returns non-enumerable property names. However, this list will not include any Symbol property names, as those are treated as special locations on the object. Object.getOwnPropertySymbols(..) returns all of an object's Symbol properties. So if you concatenate both of those lists together, you'd have all the direct (owned) contents of an object.

</details>

____
## Chapter 2: How objects work

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

____

## Chapter 3: Classy Objects




<details>
<summary></summary>
</details>

