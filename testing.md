Testing
=======

## Preferred libraries

* mocha
* chai
* sinon + sinon-chai + sinon-as-promised

But you can use anything in @prepair, as long as the tests are readable ('Foo should do bar')
and the coverage is good. Tap, tape, jasmine, the builtin assert library etc.

## Skeleton

* place spec files next to the code for convenience
* describe
* it should

```js
/*eslint object-property-newline:0, import/no-named-as-default-member:0, no-invalid-this:0*/
import { expect } from 'chai';
import * as Dog from './dog';

describe('animals/dog', () => {
  describe('bite', () => {
    let bite = Dog.bite;
    let person;

    beforeEach(function () {
      person = { type: 'burglar' };
    });

    it('should bite the burglar', () => {
      let result = bite(person);
      expect(result).to.be.true;
    });

    it('should not bite the postman', () => {
      person.type = 'postman';
      let result = bite(person);
      expect(result).to.be.false;
    });
  });
});
```

## Running tests

* `npm run test`
* `npm run test:watch` - preferably with the min reporter

The file **watcher** can only watch the changes in the files that existed during the
launching of the watcher. It will ***NOT watch newly created files**.

Initialize babel in **mocha.opts**, add test setup and whitelist. Currently mocha's
recursive reader is buggy (hence the pyramid pattern):

```
--require babel-core/register
app/test/setup.js
app/**/*.spec.js
app/**/**/*.spec.js
app/**/**/**/*.spec.js
app/**/**/**/**/*.spec.js
```

A simple **setup.js** should at least initialize sinon.

```js
const sinon = require('sinon');
const mocha = require('mocha');

beforeEach(function () {
  this.sandbox = sinon.sandbox.create();
});

afterEach(function () {
  this.sandbox.restore();
});
```

These are _global beforeEach_ and _afterEach_ hooks. All scripts are run in a single context, they are NOT
run parallel (initializing the test environment for every testfile would possibly be an overkill).

**Always clean up** after a test. Do not assume that someone will do this for you (especially not in a fake
browser context)!

## Skipping tests, exclusive test

* `xit`, `it.skip`, `it.only`
* `describe.skip`, `describe.only`

As of mocha 3.0 describe strings may not have to be unique anymore.

## Chai rules

Some example chains (the ones I use most often):

```
expect(result).to.be.true;
expect(result).to.be.undefined;
expect(loggly.install).to.be.a('function');
expect(error).to.be.an.instanceOf(Error);
expect(result).to.be.empty;
expect(insta.find('li')).to.have.length(3);
```

All the chai chains: http://chaijs.com/api/bdd/

Try to avoid simple equals. Readability is important for a test. A test should be a "specification" of sorts.

* GOOD: `expect(result).to.have.length(42);` :thumbsup:
* BAD: `expect(result.length).to.equal(42);` :thumbsdown:

Always try to read your tests: _"Let's describe submitCar. It should post data to the backend api.
Expect a stub to have been called. Expect the first argument of the first call to be 'api/car'."_ etc.

In chai there are two equals:

* `eql` = deep equal, use it with objects and arrays
* `equal` = shallow equal (`===`)

## Sinon

Sinon is a spying framework. Use it to see if a function has been called. You can replace existing functions
with stubs.

### Spies

Use a spy if you want the original functionality intact.

```js
beforeEach(function () {
  this.sandbox.spy(window.location, 'replace');
  this.sandbox.spy(SeatUtils, 'resetSeat');
});

describe('go', () => {
  let go = router.go;

  it('should navigate to new hashurl _with_ history change', () => {
    go('/foo');
    expect(window.location.replace).to.have.not.been.called;
...
```

Expect(stuff).to.have.been.called = sinon chai chain.
[Nice cheatsheet](http://ricostacruz.com/cheatsheets/sinon-chai.html).

### Stubs

Stubs can be programmed via sinon, or manually (for a simple return value).

Some nice stubs:

* `tStub = this.sandbox.stub(Vue, 't', s => s);`
* `lsGetStub = this.sandbox.stub(localStorage, 'getItem');` later on the behaviour: `lsGetStub.returns(null);`
* `postStub = this.sandbox.stub(Vue.http, 'post');`
* `this.sandbox.stub(Date, 'now').returns(1468939462280); // 2016-07-19`
* `this.sandbox.stub(moment.prototype, 'format', () => 'Thursday');`

Some nice expectations:

* `expect(postStub.firstCall.args[0]).to.equal('api/car');`
* `expect(tStub).to.have.been.calledWith('currency_symbol_GBP');`
* `expect(Vue.t.lastCall).to.be.calledWith('dog-many', [5]);`

Sandboxing; see also:

* this.sandbox
* this.create

### What to stub?

* If you stub everyting = unit test.
* If you stub nothing = integration test.

```
 unit | ------------------------------------------ | integ - - - - - - - > e2e
```

Tests are for

1. specification (think about reading a spec file)
2. helping you write your code (running a test watcher, writing your test and your function together)
3. seatbelt and airbag, in case of refactoring

If you need a helping hand during development, feel free to write extra tests. If you want to
delete them in the end (having one reliable unit test), it is up to. Tests come and go, they are
modified, rewritten, deleted, added all the time.

## Stubbing external dependencies

* node context: wrapper object (`index.js`)
* node context: [proxyquire](https://github.com/thlorenz/proxyquire)
* es6 imports via babel: `* as`

Es6 * default example:

```
import * as monthlyPricesWrapper from './monthly-prices';
describe('/utils/fare-finder/all-days', () => {
    let monthlyPricesStub;
    beforeEach(function() {
        monthlyPricesStub = this.sandbox.stub(monthlyPricesWrapper, 'default');
    });
```

BUT: es6 imports should be [immutable](http://www.2ality.com/2017/01/babel-esm-spec-mode.html)!
If babel fixes this, this will no longer work. :bomb:

Possible solutions:

* [webpack inject loader](https://github.com/plasticine/inject-loader)?
* [proxyquire universal](https://github.com/bendrucker/proxyquire-universal)?
* we already wrap the require function, maybe we can come up with a clever hack

## Testing private code

Private methods should **not** be tested per definitionem, but I usually expose them for my own sake.
While this may not be nice, having a private exposed and tested is better than having no tests at all.

The final **public** function shall be tested, but if you covered all the privates for your own
amusement, you may relax a bit with the final one. This is not nice, but again, write tests because
they help you, not because someone forced you to do so.

## Testing big objects (input, output)

Complex javascript objects are a pain in the back to test. Possible strategies:

1. try to **modularize** your code, create small components, pure functions
2. use a **.mock file** next to your .spec file. Store mock input jsons there. Do NOT store the expectations
   in the mock file, expectations should go into the test file.
3. test your **private methods**. It's better than nothing.
4. simplyify the expectation: `expect(result.map(item => item.id).sort()).to.eql([1, 2, 3, 4, 5]);`,
   but do not forget: **fuzzy tests are dangerous**.

Technically one **should** deep compare even complex, big result object, so the smallest change shall break
the test. If you are not doing this, your test is fuzzy and may be slightly evil, or borderline useless.

Try to find a good balance between maintainability and fuzzyness if you must.

Big schema changes may require you to throw away or regenerate your mock data.
Do not be afraid to fully rewrite tests.

## Browser context

At @prepair try to avoid the browser context (your target should be the current (!) node context).

If you do need a browser context, use `jsdom`.

In **test/setup.js** create a mock context for anything **you need**, nothing more.

Example hacks are [here](https://gist.github.com/szkrd/d4e922f868d02680f249910c9a39c305).

At minimum you have to use the node global object:

```js
const virtualConsole = jsdom.createVirtualConsole();
virtualConsole.on('jsdomError', error => console.error(error.stack, error.detail));
const document = jsdom.jsdom('<!doctype html><html><body></body></html>', {
    virtualConsole,
    features: { FetchExternalResources: false, ProcessExternalResources: false }
});
const window = document.defaultView;
global.document = document;
global.window = window;
```

## Testing Vue viewmodels

* so far we can test simple components in a headless environment.
* for observable/reactive values special setup is needed
* see: `vue-create-headless-component`.

Example:
```js
it('should get a location by its internal code', () => {
  let title = create(TitleComponent, { isReturn: false }, storeState);
  //                  ^component       ^                   ^
  //                                   ^params             ^
  //                                                       ^vuex mock state
  expect(title.getLocationByCode('FOO').shortName).to.equal('Foo City');
});
```

## Testing promises

* you may want to use `sinon-as-promised`: `myStub.resolves(foo);`
* consider `co-mocha`: yield/async, catch
* if you are not using co or other async helper, do not forget to call `done`

```
it('should do stuff', function (done) {
  getFoo().then(() => {}).catch(err => {
    expect(err).to.be.instanceOf(VeryFooError);
    done();
  });
});
```
