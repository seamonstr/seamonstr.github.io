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

## Functions

function myFunction(a: int)