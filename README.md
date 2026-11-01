# `nubx` stops before running anything, because of a tsconfig inside `node_modules`

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

On Windows:

```sh
npm install
npx hello
nubx hello
```

That is where I first hit this. But it is not Windows-specific — on Ubuntu the same thing happens when the entry is spelled through the link:

```sh
nub node_modules/@repro/cli/bin.cjs     # stops
nub packages/cli/bin.cjs                # runs fine
```

## What happened

`npx hello` prints the line and exits 0. `nubx hello` exits 1, and nothing from my program is printed, so it seems to stop before running it:

```
Nub: G:\Projects\npx-works-nubx-errors\node_modules\@repro\cli\tsconfig.json: File '../../tsconfig.base.json' not found.
Error: the project's tsconfig.json could not be read in full (see above).
Fix the config, or use --node to run without Nub's TypeScript features, path aliases included.
```

On Ubuntu it is the same error, with `/root/npx-works-nubx-errors/node_modules/@repro/cli/tsconfig.json` as the path.

My guess for why `nubx` only shows it on Windows: npm writes `node_modules\.bin\hello.cmd` there, and that shim spells the target as `node_modules\@repro\cli\bin.cjs`, so the entry keeps the link path. On Linux `.bin/hello` is a symlink and nub runs it as `node_modules/.bin/hello`, so `nubx hello` happens to work and only the explicit linked path reproduces.

## What I tried

- `--node` works.
- Adding `nub.jsonc` with `{ "tsconfig": "./tsconfig.base.json" }` works.
- Adding a `tsconfig.json` in the repo root does not help — same error.

## What I don't understand

1. `bin.cjs` is plain CommonJS. The run never loads a `.ts` file, so why does it need to read a tsconfig at all?
2. The file in the error is inside `node_modules` and belongs to my own package, but the message calls it "the project's tsconfig.json". Is a config found there meant to be the project config?
3. `../../tsconfig.base.json` is correct from `packages/cli/`, it is only wrong when resolved from `node_modules/@repro/cli/`. Is that expected for linked workspace packages?
4. Is this intended? If so, what is the setup you would recommend — `nub.jsonc`, a different location for the tsconfig, or something else?

Environment: nub v0.9.7. Windows 10, node v26.13.0, npm 12.0.2. Ubuntu 24.04, same nub version.
