# Symbol

`Symbol` is a built-in object whose constructor returns a `symbol` primitive - also a called a **Symbol value** or just a **symbol** - that's guaranteed to be unique. Symols are often used to add unique property keys to an object that won't collide with keys any other code might add to the object, and which are hidden from any mechanisms other code will typically use to access the object. That enables a form of weak encapsulation, or a weak form of information hiding. That enables a form of weak encapsulation, or a weak form of information hiding. 

Every `Symbol()` call is guaranteed to return a unique Symbol. Every `Symbol.for("key")` call will always return the same Symbol for a given value of `"key"`. When `Symbol.for("key")` is called, if a Symbol with the given key can be found in the global Symbol registry, that Symbol is returned. Otherwise, a new Symbol is created, added to the global Symbol registry under the given key, and returned.

## Description

To create a new primitive Symbol, you write `Symbol()` with an optional string as its description:

```js
const sym1 = Symbol();
const sym2 = Symbol('foo');
const sym3 = Symbol('foo');
```

The above code creates three new Symbols. Note that `Symbol("foo")` does not coerce the string "foo" into a Symbol. It creates a new Symbol each time:

```js
Symbol('foo') === Symbol('foo'); // false
```

The following syntax with the `new` operator will throw a `TypeError`:

```js
const sym = new Symbol(); // TypeError
```

This prevents authors from creating an explicit `Symbol` wrapper object instead of a new Symbol value and might be surprising as creating explicit wrapper objects around primitive data types is generally possible (for example, `new Boolean`, `new String`, `new Number`).

If you really wnat to create a `Symbol` wrapper object, you can use the `Object()` function:

```js
const sym = Symbol('foo');
typeof sym; // 'symbol'
const symbObj = Object(sym);
typeof symObj; // 'object'
```

Because symbols are the only primitive data type that has reference identity (that is, you cannot create the same symol twice), they behave like objects in some way.
For example, they are garbage collectable and can therefore be stored in `WeakMap`, `WeakSet`, `WeakRef`, and `FinalizationRegistry` objects.

### Shared Symbols in global Symbol registry

The above syntax using the `Symbol()` function will create a Symbol whose value remains unique throughout lifetime of the program.
To create Symbol available across files and even across realms (each of which has its own global scope), use the methods `Symbol.for()` and `Symbol.keyFor()` to set and retrieve Symbols from the global Symbol registry.

Note that "global Symbol registry" is only a fictitious concept and may not correspond to any internal data structure in the JavasScript engine - and even if such a registry exists, its content is not available to the JavaScript code, except through the `for()` and `keyFor()` methods.

The method `Symbol.for(tokenString)` takes a string key and returns a symbol value from the registry, while `Symbol.keyFor(symbolValue)` takes a symbol value and returns the string key corresponding to it.
Each is the other's inverse, so the following is `true`:

```js
Symbol.keyFor(Symbol.for('tokenString')) === 'tokenString'; // true
```

Because registered symbols can be arbitrarily created anywhere, they behave almose exactly like the strings they wrap. Therefore, they are not guaranteed to be unique and are not garbage collectable. Therefore, registered symbols are disallowed in `WeakMap`, `WeakSet`, `WeakRef`, and `FinalizationRegistry` objects.

### Well-known Symbols

All static properties of the `Symbol` constructor are Symbols themselves, whose values are constant across realms.
They are known as well-knwon Symbols, and their purpose is to serve a "protocols" for certain built-in JavaScript operations,
allowing users to customize the language's behavior.
For example, if a constructor function has a method with `Symbol.hasInstance` as its name, this method will encode its behavior with the `instanceof` operator.

