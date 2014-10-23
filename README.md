#Mock Promises
Mock Promises is a library for synchronously testing asynchronous javascript promises.  It is designed to feel similar to libraries for synchronously testing asynchronous http requests, such as [jasmine-ajax].

##Supported Libraries
Mock Promises currently supports the [Q] promise library and native Promises in Chrome. More coming soon. If you would like to use Mock Promises for a library that is not supported, please open a github issue.

Mock Promises is test framework agnostic, and we have code examples for the [jasmine] and [mocha] testing frameworks in the [spec/javascripts] directory.

##Installation
#### Self Vendoring

Download [mock-promises.js] and add it to your project.  If you are using the jasmine gem, mock-promises.js needs to be in the src_files path in jasmine.yml.

#### NPM
In node, you can use `npm install mock-promises`.  

Once the module is installed `require('mock-promises')` in your specs will attach `mockPromises` to the global namespace.

Node does not currently support native promises and you may need to use the [es6 promise polyfill] if you want to run the example specs.

## Q
#### Setup
(These directions are for the Q library, or any libraries with a similar re-use of the `then` function.)

To start mocking, use the `install` function.  The argument to `install` is the `Promise` class used by your promise library.  It happens to be `Q.makePromise` for Q.

```js
mockPromises.install(Q.makePromise)
```

It is recommended to put this is in the global `beforeEach` of your spec helper.  Any promises that are instantiated before you start mocking will not be mocked.

To prevent test pollution, you should reset mocking between tests
```js
mockPromises.reset();
```

####Teardown
To turn off mocking, use the `uninstall` function

```js
mockPromises.uninstall();
```

## Native Promises
If you are using Native promises, mock promises needs to mock out the constructor, which requires `getMockPromise`.  This method is doing a lot more than `install`, and is still in development.

```js
Promise = mockPromises.getMockPromise(Promise);
```

to turn off mock in this case, there is a `getOriginalPromise` method

```js
Promise = mockPromises.getOriginalPromise();
```

###Promise Resolution Policy
Promises often lead to other promises, for example, `promise.then(function1).then(function2)`, so we had to decide what happens to the `function2` on a promise when you execute the `function1`. We have chosen to force the user to explicitly ask for each callback to be executed, so that they do not accidentally execute callacks without realizing it. Even `executeForResolvedPromises` will only go down one level of the chain for each call. To manually go down a promise chain, you can use `iterateForPromise`.

##API

### install(PromiseClass)
Starts mocking promises of the given Promise Class

### uninstall()
Stops mocking promises mocked by `install`

### reset()
Resets all [Contracts].

### getMockPromise(PromiseClass)
Returns a mocked version of PromiseClass; needed for mocking native promises

### getOriginalPromise
Returns the unmocked version of PromiseClass mocked by `getMockPromise`; needed for unmocking native promises

###executeForPromise(mockedPromise)
Executes all fulfillmentHandlers if the mocked promise is resolved. Executes all rejectionHandlers if the mocked promise is rejected. Will not execute handlers that have already been executed.

###executeForPromises(mockedPromises)
Calls `executeForPromise` on each mocked promise in the array of mocked promises, in order.

###iterateForPromise(mockedPromise)
In the event of a tree of promises created by chaining `then` off of mockedPromise, this will go down the tree and find the first level that has not yet been exectued and then execute it. If top-level callbacks on mockedPromise have not been executed, this has the same effect as `executeForPromise`. If the entire tree has already been executed, nothing happens.

###iterateForPromises(mockedPromises)
Calls `iterateForProimse` on each mocked promise in the array of mocked promisees, in order.

###valueForPromise(mockedPromise)
Returns the resolved value of the mocked promise without executing any of its callbacks.

##<a id="Contracts"></a>Contracts

Everytime `then` is called on a mocked promise, it generates a `contract` within mock promises.  A contract represents a promise and a set of handlers.  These are mostly used internally, but can be useful for debugging purposes.  

### contracts.all()
Returns an array of all available contracts.

### contracts.forPromise(mockedPromise)
Returns an array of all contracts associated with the mocked promise. 

##Examples

To see more detailed examples, look in [mock-promises_spec.js]. Some examples are included below.

```js
describe("executeForPromise", function() {
  var promise1, promise2;
  beforeEach(function() {
    mockPromises.install(Q.makePromise);
    mockPromises.reset();
    promise1 = Q("foo");
    promise2 = Q("bar");
  });
  it("calls handlers for that promise synchronously", function() {
    var promisedValue;
    promise1.then(function(value) {
      promisedValue = value;
    });
    promise2.then(function(value) {
      promisedValue = value;
    });
    promisedValue = "not foo";
    mockPromises.executeForPromise(promise1);
    expect(promisedValue).toEqual("foo");
  });
});

describe("iterateForPromise", function() {
 it('calls the next generation of handlers if the promise has been executed', function() {
    var parentValue = 'not foo';
    promise1 = PromiseWrapper('foo');
    promise2 = promise1.then(function(value) {
      parentValue = value;
      return value + 'bar';
    });
    var childValue1 = 'not foobar'
    var childValue2 = 'not foobar';
    promise2.then(function(value) {
      childValue1 = value;
    });
    promise2.then(function(value) {
      childValue2 = value;
    });
    mockPromises.executeForPromise(promise1);
    expect(parentValue).toEqual("foo");
    expect(childValue1).toEqual("not foobar");
    expect(childValue2).toEqual("not foobar");
    mockPromises.iterateForPromise(promise1);
    expect(parentValue).toEqual("foo");
    expect(childValue1).toEqual("foobar");
    expect(childValue2).toEqual("foobar");
  });
});

```

[Contracts]:#Contracts
[jasmine]:https://github.com/pivotal/jasmine
[mocha]:https://github.com/visionmedia/mocha
[spec/javascripts]:https://github.com/charleshansen/mock-promises/tree/master/spec/javascripts
[jasmine-ajax]:https://github.com/pivotal/jasmine-ajax
[mock-promises.js]:https://github.com/charleshansen/mock-promises/blob/master/lib/mock-promises.js
[mock-promises_spec.js]:https://github.com/charleshansen/mock-promises/blob/master/spec/javascripts/mock-promises_spec.js
[Q]:https://github.com/kriskowal/q
[RSVP]:https://github.com/tildeio/rsvp.js/
[es6 promise polyfill]:https://github.com/jakearchibald/es6-promise

