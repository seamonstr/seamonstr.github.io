---
title: Getting to grips with Typescript
---

Typescript is a type-safe version of Javascript. It transpiles to old-fashioned ES5 by default, so that old browsers don't bork. The idea is that TypeScript can support all the brand new ES features without you having to stress about if the in-play browsers can support them.

Also, it does some non-JS-supported stuff like generics (duh, given JS has no typing), interfaces, ...

Install with npm. Note that `npm init` creates  a package.json for you.

This gives you a compiler, `tsc`:

```bash
# Generates 'hello_world.js'
npx tsc hello_world.ts
```

Even if compilation fails you get a JS file, with no typing information in it.  You can turn this behaviour off with `--noEmitOnError`.

## Basic typing

With typescript, types will be inferred by how you intitialise a variable:
```typescript
var a = "boo"; // Inferred as type string
a = 10; // Fail - mismatched type
```

You can also explicitly type them:
```typescript
var a : string = "boo";
```

Declaring a var of type "any" gives you freeform; it's not idiomatic, though. Mainly used for backward-compatibility.

```typescript
var a : any = "boo";
a = 10; // fine
```

## Functions & classes

Typing information is added to functions:

```js
function myFunction(a: int): string {
	return `Yes, it's ${a}!`
}
```

And to classes:

```js
class MyClass {

	_value: int;

	constructor(value: int) {
		this._value = value;
	}

	getValue: int() {
		return _value;
	}
}
```

### Generics

Generics work as you'd expect. They're only used for type checking, so have no impact on the generated
JavaScript.

```js
class MyClass<T> {
    _thing: T;

    constructor(thing: T) {
        this._thing = thing;
    }

    getThing(): T {
        return this._thing;
    }
}

function genericFunc<T>(a: T, b: T) {
    return a == b;
}
```

## Enums

Enums in TypeScript look as follows:

```js
const enum MyStuff {
    apple = "Apple",
    orange = "Orange"
}

function getThings(o: MyStuff): string {
    if (o == MyStuff.apple) {
        return MyStuff.apple
    } else {
        return MyStuff.orange
    }
}
```

The generated JS just contains the string constants everywhere an enum name is referenced. Declaring `number` values for the enums results in just the numbers being present.

## Imports

TypeScript uses the ES6 import and export syntax.

The generated javascript will contain `commonJS`-style imports and exports by default, though can you can make it `AMD` modules by passing `--module amd` to `tsc`.

Remember that you don't need to specify the `.js` extension on imports in any of the import schemes.

### Importing legacy JS files into TS

Many libraries (jQuery, for example) are included in the browser in their own `script` tags; you don't bundle them with your code.

However, Typescript still needs their type info and their interfaces to be able to validate your code. 

To this end, most libraries ship their type definitions as separate artifacts to the main JS libraries, with a naming convention of `@types/<LIB_NAME>`.  jQuery's, for example, is `@types/jquery`.  The types themselves are in TS files with a naming convention of "<file>.d.ts".

You add it with `npm install @types/jquery`, at which point your TS code will compile. The full version of the lib then needs referencing from your HTML with a `script` tag, and including in your shipping website.

### tsc configuration at project level

`tsc --init` will generate a default `tsconfig.json`, which is the config file for `tsc`.  When you have a `tsconfig.json` in a folder, running `tsc` in that folder will result in all `.ts` files being transpiled.