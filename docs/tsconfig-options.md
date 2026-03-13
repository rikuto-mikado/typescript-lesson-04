# TypeScript Compiler Options Reference

A reference for `tsconfig.json` options — what they do, why they matter, and what happens without them.

---

## File Layout

### `rootDir`
Specifies the root directory of input source files. TypeScript uses this to mirror the directory structure in `outDir`.

```json
"rootDir": "./src"
```

Without it, TypeScript infers the root from your source files, which can produce unexpected output paths.

---

### `allowJs`
Allows `.js` and `.jsx` files to be compiled alongside `.ts` files.

```json
"allowJs": true
```

Useful during migration from JavaScript to TypeScript. Without it, `.js` files are ignored by the compiler.

---

### `checkJs`
Enables type checking on `.js` files (requires `allowJs`).

```json
"checkJs": true
```

Errors are reported in `.js` files as if they were TypeScript. Off by default — useful when you want gradual typing in a JS codebase.

---

### `outDir`
Redirects all compiled output (`.js`, `.d.ts`, `.map`) to the specified directory.

```json
"outDir": "./dist"
```

Without it, output files are placed next to source files.

---

## Environment Settings

### `module`
Specifies the module system for emitted JS code.

```json
"module": "nodenext"
```

| Value | Use case |
|---|---|
| `commonjs` | Node.js (legacy) |
| `nodenext` | Node.js with native ESM (`package.json` `"type"` field respected) |
| `esnext` | Bundlers (Vite, webpack, etc.) |
| `node16` | Like `nodenext` but pinned to Node 16 semantics |

`nodenext` requires `.js` extensions in import paths and respects `package.json` `"type": "module"`.

---

### `target`
Sets the JavaScript language version for emitted output.

```json
"target": "esnext"
```

Controls which syntax is downleveled (e.g., `async/await`, optional chaining). With `esnext`, no downleveling — output matches modern JS.

---

### `types`
Limits which `@types/*` packages are included globally.

```json
"types": []
```

Empty array means **no automatic global type injection** from `node_modules/@types`. You must explicitly import types. Prevents accidental leakage of e.g. `@types/node` globals into browser code.

---

### `lib`
Specifies built-in API type definitions to include (DOM, ES2020, etc.).

```json
// "lib": ["esnext"]
```

When omitted, TypeScript infers a default set based on `target`. For Node.js projects, you typically set `["esnext"]` and exclude DOM types.

---

## Other Outputs

### `sourceMap`
Generates `.js.map` files that map compiled JS back to TypeScript source.

```json
"sourceMap": true
```

Required for debuggers and stack traces to point to `.ts` files instead of compiled `.js`.

---

### `declaration`
Generates `.d.ts` type declaration files alongside compiled JS.

```json
"declaration": true
```

Essential for libraries — consumers get type information without needing your source. Required when publishing to npm.

---

### `declarationMap`
Generates `.d.ts.map` files that map declarations back to source `.ts`.

```json
"declarationMap": true
```

Enables "Go to Definition" in editors to jump to the original `.ts` source instead of the `.d.ts` file. Only useful if source is distributed alongside declarations.

---

### `noEmitOnError`
Suppresses all output file generation if any type errors are reported.

```json
"noEmitOnError": true
```

Without it, TypeScript emits JS even when there are type errors. With it, a broken build produces no artifacts — prevents deploying type-unsafe code. Particularly important in CI/CD pipelines.

---

## Stricter Typechecking Options

### `noUncheckedIndexedAccess`
Adds `undefined` to the type of any index signature access.

```json
"noUncheckedIndexedAccess": true
```

```ts
const arr: string[] = ["a", "b"];
const x = arr[0]; // type: string | undefined  (not just string)
```

Prevents runtime errors from out-of-bounds array/object access that TypeScript would otherwise consider safe.

---

### `exactOptionalPropertyTypes`
Distinguishes between a property being absent and a property explicitly set to `undefined`.

```json
"exactOptionalPropertyTypes": true
```

```ts
interface Config {
  timeout?: number;
}

const c: Config = { timeout: undefined }; // Error — can omit, but can't explicitly set undefined
```

Without it, `{ timeout: undefined }` and `{}` are treated the same. Catches subtle bugs in optional property handling.

---

## Style Options (currently commented out)

### `noImplicitReturns`
Errors when a function has code paths that don't return a value.

```ts
function getName(id: number): string {
  if (id === 1) return "Alice";
  // Error: Not all code paths return a value
}
```

---

### `noImplicitOverride`
Requires the `override` keyword when overriding a method in a subclass.

```ts
class Animal {
  speak() {}
}
class Dog extends Animal {
  override speak() {}  // must use `override`
}
```

Prevents accidentally overriding a method when the base class renames or removes it.

---

### `noUnusedLocals`
Errors on declared local variables that are never read.

---

### `noUnusedParameters`
Errors on function parameters that are never used. Prefix with `_` to suppress: `_unused`.

---

### `noFallthroughCasesInSwitch`
Errors on `switch` cases that fall through to the next case without a `break` or `return`.

---

### `noPropertyAccessFromIndexSignature`
Requires bracket notation for properties that come from index signatures.

```ts
interface Env {
  [key: string]: string;
}
const e: Env = {};
e.FOO;       // Error — must use e["FOO"]
e["FOO"];    // OK
```

---

## Recommended Options

### `strict`
Enables a collection of strict type-checking flags as a group:

| Flag | What it does |
|---|---|
| `strictNullChecks` | `null`/`undefined` are not assignable to other types |
| `strictFunctionTypes` | Stricter checking of function parameter types |
| `strictBindCallApply` | Correct types for `bind`, `call`, `apply` |
| `strictPropertyInitialization` | Class properties must be initialized in constructor |
| `noImplicitAny` | Error on expressions with implicit `any` type |
| `noImplicitThis` | Error on `this` with implicit `any` type |
| `useUnknownInCatchVariables` | `catch` clause variables typed as `unknown` instead of `any` |
| `alwaysStrict` | Emits `"use strict"` and parses in strict mode |
| `forceConsistentCasingInFileNames` | Disallows casing-inconsistent imports (important on case-insensitive filesystems like macOS) |

---

### `jsx`
Controls how JSX is transformed.

```json
"jsx": "react-jsx"
```

| Value | Output |
|---|---|
| `react` | `React.createElement(...)` — requires `import React` |
| `react-jsx` | Uses new JSX transform (React 17+) — no import needed |
| `preserve` | Leaves JSX as-is for a bundler to handle |

---

### `verbatimModuleSyntax`
Enforces that `import type` is used for type-only imports, and removes them at emit.

```json
"verbatimModuleSyntax": true
```

```ts
import type { Foo } from './foo'; // OK — erased at emit
import { Foo } from './foo';      // Error if Foo is only a type
```

Prevents issues where bundlers or runtimes emit side-effectful imports that were meant to be type-only.

---

### `isolatedModules`
Ensures each file can be safely transpiled in isolation (without full type information).

```json
"isolatedModules": true
```

Required for tools like Babel, esbuild, or SWC that transpile file-by-file. Flags constructs that require cross-file type knowledge (e.g., `const enum`, re-exported types without `type` keyword).

---

### `noUncheckedSideEffectImports`
Errors on imports that only exist for their side effects if the module can't be resolved.

```json
"noUncheckedSideEffectImports": true
```

```ts
import './setup'; // Error if './setup' doesn't exist or can't be resolved
```

---

### `moduleDetection`
Controls how TypeScript decides if a file is a module or a script.

```json
"moduleDetection": "force"
```

| Value | Behavior |
|---|---|
| `auto` | Files with `import`/`export` are modules; others are scripts |
| `force` | All files treated as modules regardless of content |
| `legacy` | Old behavior — based on `module` setting |

`force` avoids accidental global scope leakage from files that have no imports/exports yet.

---

### `skipLibCheck`
Skips type checking of all declaration files (`.d.ts`).

```json
"skipLibCheck": true
```

Significantly speeds up compilation. Suppresses errors from buggy or conflicting type definitions in `node_modules`. Almost always safe to enable — you trust your dependencies' published types.

---

### `esModuleInterop`
Adds helper code to allow default-importing CommonJS modules.

```ts
import fs from 'fs'; // Works with esModuleInterop, requires * as without it
```

**Not needed with `"module": "nodenext"`** — native ESM handles interop differently.

---

### `forceConsistentCasingInFileNames`
Already covered above — included in `strict`.

---

## Notes

- Options marked with `//` in `tsconfig.json` are commented out and inactive.
- `strict: true` is a superset — enabling individual flags it covers is redundant.
- For a full reference: https://aka.ms/tsconfig
