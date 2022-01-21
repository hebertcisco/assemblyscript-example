# Introduction

AssemblyScript compiles a **variant** of [TypeScript](https://www.typescriptlang.org) \(a typed superset of JavaScript\) to [WebAssembly](https://webassembly.org) using [Binaryen](https://github.com/WebAssembly/binaryen), looking like:

```ts
export function fib(n: i32): i32 {
  var a = 0, b = 1
  if (n > 0) {
    while (--n) {
      let t = a + b
      a = b
      b = t
    }
    return b
  }
  return a
}
```

In its simplest form, it is JavaScript with [WebAssembly types](https://www.assemblyscript.org/types.html), compiled statically to a bunch of WebAssembly [exports and imports](https://www.assemblyscript.org/exports-and-imports.html), like so:

```sh
asc fib.ts --binaryFile fib.wasm --optimize
```

As such, it differs from running dynamically typed JavaScript (just in time) in that it instead [statically compiles](https://www.assemblyscript.org/compiler.html) to a strictly typed WebAssembly binary **ahead of time**, quite similar to what a traditional compiler would do. However, since it is deliberately designed to be very similar to JavaScript, it has the potential to integrate seamlessly with existing Web Platform concepts to produce lean and mean WebAssembly binaries, while also making it almost natural to use for those who are already familiar with writing code for the Web.

## Low-level perspective

AssemblyScript provides WebAssembly and compiler foundations as [low-level built-ins](https://www.assemblyscript.org/stdlib/builtins.html), making it suitable as a thin layer on top of raw WebAssembly. In fact, its standard library is implemented on top of just these foundations.

For example, memory can be accessed using the `load<T>(offset[, immOffset])` and `store<T>(offset, value[, immOffset])` built-ins that compile to WebAssembly instructions directly:

```ts
store<i32>(ptr, load<i32>(ptr) + load<i32>(ptr, 4), 8)
```

For comparison, the following C code is roughly equivalent:

```c
*(ptr + 2) = *ptr + *(ptr + 1)
```

## High-level perspective

On top of its low-level capabilities, AssemblyScript provides a JavaScript-like [standard library](https://www.assemblyscript.org/stdlib/globals.html) that is closely integrated with [memory management](/memory.md) and [garbage collection](./garbage-collection.md). The standard library provides implementations of many of the classes and namespaces one would expect from JavaScript, including [Math](./stdlib/math.md) (double and single precision), [Array](./stdlib/array.md), [String](./stdlib/string.md), [Map](./stdlib/map.md), [Typed arrays](./stdlib/typedarray.md) and so on.

The example above could look like this in idiomatic AssemblyScript:

```ts
var view = new Int32Array(12)
...
view[2] = view[0] + view[1]
```

## Setting realistic expectations

[Strictly typed](https://www.assemblyscript.org/basics.html#strictness) AssemblyScript differs a bit from idiomatic TypeScript, which even though typed, attempts to describe all the dynamic features of JavaScript, many of which cannot be efficiently compiled ahead of time. Not a blocker, but takes a bit to get used to.

Also, we are patiently waiting for [future WebAssembly features](./status.md) (marked as 🦄 throughout the documentation) to become available for us to use. In particular, we are looking forward to proposals that promise to improve WebAssembly's integration with the Web Platform, e.g. to more naturally and efficiently share strings, arrays and objects with JavaScript.

# Quick Start

Paving the way to what might be your first AssemblyScript module.

## Prerequisites

The following assumes that a [recent version of Node.js](https://nodejs.org) and its package manager [npm](https://www.npmjs.com) \(comes with Node.js\) are installed, with the commands below executed in a command prompt. Basic knowledge about writing and working with TypeScript modules, which is very similar, is a plus.

## Setting up a new project

To get started with AssemblyScript, switch to a new directory and initialize a new node module:

```sh
npm init
```

Now install both the [loader](./loader.md) and the [compiler](./compiler.md) using npm. Let's assume that the compiler is not required in production and make it a development dependency:

```sh
npm install --save @assemblyscript/loader
npm install --save-dev assemblyscript
```

::: tip
If you need a [specific version](https://github.com/AssemblyScript/assemblyscript/releases) of the loader and/or the compiler, append the respective version number as usual.
:::

Once installed, the compiler provides a handy scaffolding utility to quickly set up a new AssemblyScript project, for example in the directory of the just initialized node module:

```sh
npx asinit .
```

It automatically creates the recommended directory structure and configuration files:

```
This command will make sure that the following files exist in the project
directory '/path/to/mymodule':
  ./assembly
  Directory holding the AssemblyScript sources being compiled to WebAssembly.
  ./assembly/tsconfig.json
  TypeScript configuration inheriting recommended AssemblyScript settings.
  ./assembly/index.ts
  Example entry file being compiled to WebAssembly to get you started.
  ./build
  Build artifact directory where compiled WebAssembly files are stored.
  ./build/.gitignore
  Git configuration that excludes compiled binaries from source control.
  ./index.js
  Main file loading the WebAssembly module and exporting its exports.
  ./tests/index.js
  Example test to check that your module is indeed working.
  ./asconfig.json
  Configuration file defining both a 'debug' and a 'release' target.
  ./package.json
  Package info containing the necessary commands to compile to WebAssembly.
Do you want to proceed? [Y/n]
```

Once initialized, edit the sources in `assembly/`, tweak [compiler options](https://www.assemblyscript.org/compiler.html) in [`asconfig.json`](https://www.assemblyscript.org/compiler.html#asconfig-json) to fit your needs, and run the build command to compile your module to WebAssembly:

```sh
npm run asbuild
```

## Next steps

Once compiled, you may run the tests in `tests/index.js`:

```sh
npm test
```

add [imports](./exports-and-imports.md#imports) to the generated `index.js` (instantiates the module and re-exports it):

```js
...
const imports = {
  "assembly/index": {
    declaredImportedFunction: function(...) { ... }
  }
};
...
```

and ultimately use your WebAssembly module like a normal node module:

```js
const myModule = require("path/to/mymodule");
myModule.add(1, 2);
```

::: tip NOTE
Your module's exports only understand integers and floats for now, with strings and objects being passed as pointers, but we'll get into that later when covering the [loader](/loader.md).
:::

Read on to [learn more](https://www.assemblyscript.org/basics.html)!
