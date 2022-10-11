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
<summary>static</summary>
  
  `static` - methods/fields reside on the constructor (function object) itself (neither on the constructor prototype nor on the instance). This behaviour is useful when developers need access to some class-related data independed of any instance they may or may have created. Static property or static functions can also be called non-instance properties and instance unaware functions.

```
class Point2d {
    // class statics
    static origin = new Point2d(0,0)
    static distance(point1,point2) {
        return Math.sqrt(
            ((point2.x - point1.x) ** 2) +
            ((point2.y - point1.y) ** 2)
        );
    }
    ...
 }
 
console.log(`Starting point: ${Point2d.origin}`);
// Starting point: (0,0)

var next = new Point2d(3,4);
console.log(`Next point: ${next}`);
// Next point: (3,4)

console.log(`Distance: ${
    Point2d.distance( Point2d.origin, next )
}`);
// Distance: 5
```
  
The `Point2d.origin` is a static property, which just so happens to hold a constructed instance of our class. And `Point2d.distance(..)` is a static function that computes the 2-dimensional cartesian distance between two points.

Of course, we could have put these two somewhere other than as statics on the class definition. But since they're directly related to the `Point2d` class, it makes most sense to organize them there.
  
The value in a static initialization (`static whatever = ..`) can include `this` references, which refers to the class itself (actually, the constructor) rather than to an instance:
  
</details>


<details>
<summary>private</summary>

`private` - stores info that cannot be seen from outside the class. Private members/methods are private only to the class they're defined in, and are not inherited in any way by a subclass.

```
class Point2d {
    // statics
    static samePoint(point1,point2) {
        return point1.#ID === point2.#ID;
    }

    // privates
    #ID = null
    #assignID() {
        this.#ID = Math.round(Math.random() * 1e9);
    }

    // publics
    x
    y
    constructor(x,y) {
        this.#assignID();
        this.x = x;
        this.y = y;
    }
}

var one = new Point2d(3,4);
var two = new Point2d(3,4);

Point2d.samePoint(one,two);         // false
Point2d.samePoint(one,one);         // true
```

The `#whatever` syntax (including this.#whatever form) is only valid inside class bodies. It will throw syntax errors if used outside of a class.

Unlike public fields/instance members, private fields/instance members must be declared in the class body. You cannot add a private member to a class declaration dynamically while in the constructor method; `this.#whatever = ..` type assignments only work if the `#whatever` private field is declared in the class body. Moreover, though private fields can be re-assigned, they cannot be deleted from an instance, the way a public field/class member can.
  
Because "inheritance" in JS is sharing (through the [[Prototype]] chain), if you invoke an inherited method in a subclass, and that inherited method in turn accesses/invokes privates in its host (base) class, this works fine:

```
class Point2d { /* .. */ }

class Point3d extends Point2d {
    z
    constructor(x,y,z) {
        super(x,y);
        this.z = z;
    }
}

var one = new Point3d(3,4,5);
```
  
It's still a shame though that Point3d has no way to access/influence, or indeed even knowledge of, the #ID / #assignID() privates from Point2d:

```
class Point2d { /* .. */ }

class Point3d extends Point2d {
    z
    constructor(x,y,z) {
        super(x,y);
        this.z = z;

        console.log(this.#ID);      // will throw!
    }
}
```
  
You may want to check to see if a private field/method exists on an object instance. For example (as shown below), you may have a static function or method in a class, which receives an external object reference passed in. To check to see if the passed-in object reference is of this same class (and therefore has the same private members/methods in it), you basically need to do a "brand check" against the object.

Such a check could be rather convoluted, because if you access a private field that doesn't already exist on the object, you get a JS exception thrown, requiring ugly try..catch logic.

But there's a cleaner approach, so called an "ergonomic brand check", using the in keyword:
  
```
  class Point2d {
    // statics
    static samePoint(point1,point2) {
        // "ergonomic brand checks"
        if (#ID in point1 && #ID in point2) {
            return point1.#ID === point2.#ID;
        }
        return false;
    }

    // privates
    #ID = null
    #assignID() {
        this.#ID = Math.round(Math.random() * 1e9);
    }

    // publics
    x
    y
    constructor(x,y) {
        this.#assignID();
        this.x = x;
        this.y = y;
    }
}

var one = new Point2d(3,4);
var two = new Point2d(3,4);

Point2d.samePoint(one,two);         // false
Point2d.samePoint(one,one);         // true
```
  

</details> 

<details>
<summary>private statics</summary>
  
Static properties and functions can also use # to be marked as private:
 
```
class Point2d {
    static #errorMsg = "Out of bounds."
    static #printError() {
        console.log(`Error: ${this.#errorMsg}`);
    }

    // publics
    x
    y
    constructor(x,y) {
        if (x > 100 || y > 100) {
            Point2d.#printError();
        }
        this.x = x;
        this.y = y;
    }
}

var one = new Point2d(30,400);
// Error: Out of bounds.
```
  
The #printError() static private function here has a this, but that's referencing the Point2d class, not an instance. As such, the #errorMsg and #printError() are independent of instances and thus are best as statics. Moreover, there's no reason for them to be accessible outside the class, so they're marked private.

Remember: private statics are similarly not-inherited by subclasses just as private members/methods are not.

Gotcha: Subclassing With Static Privates and this
Recall that inherited methods, invoked from a subclass, have no trouble accessing (via this.#whatever style references) any privates from their own base class:

```
class Point2d {
    // ..

    getID() {
        return this.#ID;
    }

    // ..
}

class Point3d extends Point2d {
    // ..

    printID() {
        console.log(`ID: ${this.getID()}`);
    }
}

var point = new Point3d(3,4,5);
point.printID();
// ID: ..
```
  
That works just fine.

Unfortunately, and (to me) quite unexpectedly/inconsistently, the same is not true of private statics accessed from inherited public static functions:

```
class Point2d {
    static #errorMsg = "Out of bounds."
    static printError() {
        console.log(`Error: ${this.#errorMsg}`);
    }

    // ..
}

class Point3d extends Point2d {
    // ..
}

Point2d.printError();
// Error: Out of bounds.

Point3d.printError === Point2d.printError;
// true

Point3d.printError();
// TypeError: Cannot read private member #errorMsg
// from an object whose class did not declare it
```
  
The printError() static is inherited (shared via [[Prototype]]) from Point2d to Point3d just fine, which is why the function references are identical. Like the non-static snippet just above, you might have expected the Point3d.printError() static invocation to resolve via the [[Prototype]] chain to its original base class (Point2d) location, thereby letting it access the base class's #errorMsg static private.

But it fails, as shown by the last statement in that snippet. The reason it fails here, but not with the previous snippet, is a convoluted brain twister. I'm not going to dig into the why explanation here, frankly because it boils my blood to do so.

There's a fix, though. In the static function, instead of this.#errorMsg, swap that for Point2d.#errorMsg, and now it works:
  
```
class Point2d {
    static #errorMsg = "Out of bounds."
    static printError() {
        // the fixed reference vvvvvv
        console.log(`Error: ${Point2d.#errorMsg}`);
    }

    // ..
}

class Point3d extends Point2d {
    // ..
}

Point2d.printError();
// Error: Out of bounds.

Point3d.printError();
// Error: Out of bounds.  <-- phew, it works now!
```

If public static functions are being inherited, use the class name to access any private statics instead of using this. references. Beware that gotcha!
</details>

<details>
<summary>this vs super vs new.target</summary>

this - refers to the instance of the class if the member or method is public. If it's static, than it refers to the class constructo function itself.

super - is used if you want to access an inherited method from a subclass even if it's been overriden:
  
```
class Point2d {
    x = 3
    y = 4

    getX() {
        return this.x;
    }
}

class Point3d extends Point2d {
    x = 21
    y = 10
    z = 5

    getX() {
        return this.x * 2;
    }
    printX() {
        console.log(`x: ${super.getX()}`);
    }
}

var point = new Point3d();

point.printX();       // x: 21
```

In addition to a subclass method accessing an inherited method definition (even if overriden on the subclass) via super. reference, a subclass constructor can manually invoke the inherited base class constructor via super(..) function invocation:

```
class Point2d {
    x
    y
    constructor(x,y) {
        this.x = x;
        this.y = y;
    }
}

class Point3d extends Point2d {
    z
    constructor(x,y,z) {
        super(x,y);
        this.z = z;
    }
    toString() {
        console.log(`(${this.x},${this.y},${this.z})`);
    }
}

var point = new Point3d(3,4,5);

point.toString();       // (3,4,5)
```
  
An explicitly defined subclass constructor must call super(..) to run the inherited class's initialization, and that must occur before the subclass constructor makes any references to this or finishes/returns. Otherwise, a runtime exception will be thrown when that subclass constructor is invoked (via new). If you omit the subclass constructor, the default constructor automatically thankfully invokes super() for you.
  
new.target - is used if you may need to determine in a constructor if that class is being instantiated directly, or being instantiated from a subclass with a super() call:

</details>

## Chapter 4: This works

<details>
<summary>Default Context Invocation</summary>
  
```
var point = {
  x: null,
  y: null,

  init(x,y) {
      this.x = x;
      this.y = y;
  }
};
```

What will `this` refer to here:

```
const init = point.init;
init(3,4);
```
  
Since almost all modern JS code is being run in strict mode (ESM (ES Modules) always run in strict-mode, as does code inside a class block. And virtually all transpiled JS code (via Babel, TypeScript, etc) is written to declare strict-mode), `this` will refer to undefined here.

</details>
  
<details>
<summary>Explicit Context Invocation</summary>
  
```
var point = {
    x: null,
    y: null,

    init(x,y) {
        this.x = x;
        this.y = y;
    },
    rotate(angleRadians) { /* .. */ },
    toString() {
        return `(${this.x},${this.y})`;
    },
};

point.init(3,4);

var anotherPoint = {};
point.init.call( anotherPoint, 5, 6 );

point.x;                // 3
point.y;                // 4
anotherPoint.x;         // 5
anotherPoint.y;         // 6
```

I wanted to define anotherPoint, but I didn't want to repeat the definitions of those init(..) / rotate(..) / toString() functions from point. So I "borrowed" a function reference, point.init, and explicitly set the empty object anotherPoint as the this context, via call(..).

When init(..) is running at that moment, this inside it will reference anotherPoint, and that's why the x / y properties (values 5 / 6, respectively) get set there.

</details>

<details>
<summary>New Context Invocation</summary>
  
```
var point = {
    // ..

    init: function() { /* .. */ }

    // ..
};

var anotherPoint = new point.init(3,4);

anotherPoint.x;     // 3
anotherPoint.y;     // 4
```
  
  This example has a bit of nuance to be explained. The init: function() { .. } form shown here -- specifically, a function expression assigned to a property -- is required for the function to be validly called with the new keyword. From previous snippets, the concise method form of init() { .. } defines a function that cannot be called with new.
  
In a sense, the new keyword hijacks a function and forces its behavior into a different mode than a normal invocation. Here are the 4 special steps that JS performs when a function is invoked with new:

create a brand new empty object, out of thin air.

link the [[Prototype]] of that new empty object to the function's .prototype object (see Chapter 2).

invoke the function with the this context set to that new empty object.

if the function doesn't return its own object value explicitly (with a return .. statement), assume the function call should instead return the new object (from steps 1-3).
  
</details>

<details>
<summary>Review This</summary>
  
We've seen four rules for this context assignment in function calls. Let's put them in order of precedence:

1 Is the function invoked with new, creating and setting a new this?

2 Is the function invoked with call(..) or apply(..), explicitly setting this?

3 Is the function invoked with an object reference at the call-site (e.g., point.init(..)), implicitly setting this?

4 If none of the above... are we in non-strict mode? If so, default the this to globalThis. But if in strict-mode, default the this to undefined.

These rules, in this order, are how JS determines the this for a function invocation. If multiple rules match a call-site (e.g., new point.init.call(..)), the first rule from the list to match wins.
</details>

<details>
<summary>Lexical this in arrow functions</summary>
  
```
function outer() {
    console.log(this.value);

    // define a return an "inner"
    // function
    var inner = () => {
        console.log(this.value);
    };

    return inner;
}

var one = {
    value: 42,
};
var two = {
    value: "sad face",
};

var innerFn = outer.call(one);
// 42

innerFn.call(two);
// 42   <-- not "sad face"
```

When the innerFn(..) (aka inner(..)) function is invoked, even with an explicit context assignment via .call(..), that assignment is ignored.

I'm not sure why `=>` arrow functions even have a `call(..)` / `apply(..)` on them, since they are silent no-op functions. I guess it's for consistency with normal functions. But as we'll see later, there are other inconsistencies between regular functions and irregular `=>` arrow functions.
  
When a this is encountered (this.value) inside an => arrow function, this is treated like a normal lexical variable, not a special keyword. And since there is no this variable in that function itself, JS does what it always does with lexical variables: it goes up one level of lexical scope -- in this case, to the surrounding outer(..) function, and it checks to see if there's any registered this in that scope.

Luckily, outer(..) is a regular function, which means it has a normal this keyword. And the outer.call(one) invocation assigned one to its this.

</details>
