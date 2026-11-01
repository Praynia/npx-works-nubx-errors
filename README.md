# `npx` runs it, `nubx` stops before running anything

Hi — I'm probably missing something about how nub loads tsconfig, so I wanted to ask before calling this a bug. Sorry if this is my setup's fault.

## What I set up

A tiny npm workspace. One package, `packages/cli`, exposes a `hello` binary that prints one line. There is no TypeScript file anywhere.

```
package.json                 workspaces: ["packages/*"]
tsconfig.base.json           shared config in the repo root
packages/cli/package.json    bin: { hello: "./bin.cjs" }
packages/cli/tsconfig.json   { "extends": "../../tsconfig.base.json" }
packages/cli/bin.cjs         console.log('hello from @repro/cli')
```

## What I ran

```sh
npm install
npx hello
nubx hello
```

## What happened

`npx hello` prints the line and exits 0.

`nubx hello` exits 1, and nothing from my program is printed, so it seems to stop before running it:

```
Nub: G:\Projects\npx-works-nubx-errors\node_modules\@repro\cli\tsconfig.json: File '../../tsconfig.base.json' not found.
Error: the project's tsconfig.json could not be read in full (see above).
Fix the config, or use --node to run without Nub's TypeScript features, path aliases included.
```

## What I tried

- `nubx --node hello` works.
- Adding `nub.jsonc` with `{ "tsconfig": "./tsconfig.base.json" }` works.
- Adding a `tsconfig.json` in the repo root does not help — same error.

## What I don't understand

1. `bin.cjs` is plain CommonJS. The run never loads a `.ts` file, so why does it need to read a tsconfig at all?
2. The file in the error is inside `node_modules` and belongs to my own package, but the message calls it "the project's tsconfig.json". Is a config found there meant to be the project config?
3. `../../tsconfig.base.json` is correct from `packages/cli/`, it is only wrong when resolved from `node_modules/@repro/cli/`. Is that expected for linked workspace packages?
4. Is this intended? If so, what is the setup you would recommend — `nub.jsonc`, a different location for the tsconfig, or something else?

Environment: nub v0.9.7, node v26.13.0, npm 12.0.2, Windows 10.
