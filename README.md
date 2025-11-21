# Repro: Internal package names with nested scope (e.g. `@org/apps/mypkg`) under Pnpm / Node.js ESM

**TL;DR: we should not use it although Pnpm happens to be able to create symlink structure just fine.**

## How to test this repro

```
pnpm install
pnpm -C packages/pkg-b run test
```

Then you should see `Error [ERR_UNSUPPORTED_DIR_IMPORT]`

## Description

Npm (registry and CLI) does not support nested scoped packages such as `@org/apps/mypkg`: [nested package scopes · Issue #6333 · npm/npm](https://github.com/npm/npm/issues/6333).
However, I could not find official doc explaning what names are valid in terms of Node.js module resolution algoirthm
Interestingly, as demonstrated in this repro, Pnpm can create symlinks for "nested scoped packages" just fine with its workspace feature:

```
$ ls -lha packages/pkg-b/node_modules/@naruaway/myinternalpkg/a
lrwxr-xr-x@ 1 naru  staff    17B Nov 21 10:03 packages/pkg-b/node_modules/@naruaway/myinternalpkg/a -> ../../../../pkg-a
```

However, anyway we should not use it, as demonstrated in this repro as well, Node.js fails to resolve the package due to the following error:

```
❯ pnpm run test

> @naruaway/myinternalpkg/b@ test /Users/naru/tmp/multiple-pkgs/packages/pkg-b
> node main.js

node:internal/modules/esm/resolve:262
    throw new ERR_UNSUPPORTED_DIR_IMPORT(path, basePath, String(resolved));
          ^

Error [ERR_UNSUPPORTED_DIR_IMPORT]: Directory import '/Users/naru/tmp/multiple-pkgs/packages/pkg-b/node_modules/@naruaway/myinternalpkg/a' is not supported resolving ES modules imported from /Users/naru/tmp/multiple-pkgs/packages/pkg-b/main.js
    at finalizeResolution (node:internal/modules/esm/resolve:262:11)
    at moduleResolve (node:internal/modules/esm/resolve:859:10)
    at defaultResolve (node:internal/modules/esm/resolve:983:11)
    at #cachedDefaultResolve (node:internal/modules/esm/loader:717:20)
    at ModuleLoader.resolve (node:internal/modules/esm/loader:694:38)
    at ModuleLoader.getModuleJobForImport (node:internal/modules/esm/loader:308:38)
    at ModuleJob._link (node:internal/modules/esm/module_job:183:49) {
  code: 'ERR_UNSUPPORTED_DIR_IMPORT',
  url: 'file:///Users/naru/tmp/multiple-pkgs/packages/pkg-b/node_modules/@naruaway/myinternalpkg/a'
}
```

It looks like `@naruaway/myinternalpkg/a` is recognized as "directory import" rather than "a package" in Node.js module ESM resolution logic.
Note that I found that Node.js CJS resolution logic is loose and it can deal with this package name with nested scope.
However, we now already know one important implementation (Node.js ESM module resolution) cannot handle this case and Npm registry/CLI does not support it so we should not use this naming anyway to make sure the setup is compatible with as many tools as possible.
