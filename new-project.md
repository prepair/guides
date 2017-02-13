New project
===========

So far this is a rough guide, a set of ideas, nothing more.

## Recommended stuff

* scoped package: npm init with `--scope=prepair`
* eslint, eslint semistandard (until we make an eslint plugin from the @prepair lint style)
* mocha, comocha, chai, sinon for unit testing (use babel register in test setup or in the opts file)
* try to stay browserless, if not possible, use jsdom, phantom is a last resort
* husky for commit hooks
* standard-version for release (TODO semantic-release without CI?)
* npm tasks: dev, lint, test, test:watch, build, release
* package dist (or lib) for es2015-ie; no polyfills.
* libs that include functionality for both node and the browser MUST be transpiled,
  libs that are to be used in the node context only are fine with node6 (or current lts)
* make sure that your npm run scripts are working cross platform. If you need any executables not
  supplied with other systems (like `grep` for example), write a helper script. `cross-env` will
  not flow through `&&` command delimiters.
