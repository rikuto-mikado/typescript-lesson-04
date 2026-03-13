# TypeScript Lesson 04

## What I learned

## What was difficult

## Memo

- `tsconfig.json` is a TypeScript compiler config file, separate from `package.json` (which is for Node.js/npm). Running `npx tsc --init` generates it with options like `target`, `module`, and `strict`. The `tsc` command reads this file to compile `.ts` files.
- Running `npx tsc` compiles `.ts` files under `src/` and outputs to `dist/` based on `tsconfig.json`. With `sourceMap: true`, `declaration: true`, and `declarationMap: true`, it generates `.js`, `.js.map`, `.d.ts`, and `.d.ts.map` files per source file.
