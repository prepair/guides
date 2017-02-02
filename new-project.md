New project
===========

So far this is a rough guide, a set of ideas, nothing more.

## Recommended stuff

* scoped package: npm init with `--scope=prepair`
* eslint, eslint semistandard (until we make an eslint plugin from the @prepair lint style)
* mocha, comocha, chai, sinon for unit testing (use babel with node6 preset)
* try to stay browserless, if not possible, use jsdom, phantom is a last resort
* husky for commit hooks
* standard-version for release
* npm tasks: dev, lint, test, test:watch, build, release
* package dist (or lib) for node context, so that people with mocha and babel
  will not have a hard time testing it. No es5, bower, amd or other legacy builds.
* make sure that your npm run scripts are working cross platform. If you need any executables not
  supplied with other systems (like `grep` for example), write a helper script. `cross-env` will
  not flow through `&&` command delimiters.
