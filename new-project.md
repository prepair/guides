New project
===========

So far this is a rough guide, a set of ideas, nothing more.

## Recommended stuff

* eslint, eslint semistandard (until we make an eslint plugin from the @prepair lint style)
* mocha, comocha, chai, sinon for unit testing (use babel with node6 preset)
* try to stay browserless, if not possible, use jsdom, phantom is a last resort
* husky for commit hooks
* standard-version for release
* npm tasks: dev, lint, test, test:watch, release
* babel only for testing, publish untouched source (let webpack or future browser take care of it)
* no need to package dist (no es5, no shim, no min, no map)
