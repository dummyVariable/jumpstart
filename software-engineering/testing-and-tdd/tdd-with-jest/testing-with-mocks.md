# Mocks

Mocking is a technique to isolate test subjects by replacing dependencies with objects that you can control and inspect.

**The goal for mocking is to replace something we don’t control \(e.g. a http request which may or may not succeed\) with something we do.**

## What is a dependency?

### Library Dependency

Every system/module has its dependencies: it could be another module written by you, or it could be a module loaded from some 3rd party libraries.

Every time when you use `import` or `require` in your JavaScript code, you introduce a new `dependency` to the current JavaScript module.

### Service Dependency

If you write some frontend JavaScript codes that depends on another back-end API/web-service, you also introduce some dependency here.

## Why do we need mocking?

### Don't use mocking by default

**Mocking is a testing technique you should NOT use by default**, because there are costs associated with mocks:

* You have less confidence that your code works with the real dependency that is mocked in your tests.
* This techniques violates the principals for [black box testing](http://softwaretestingfundamentals.com/black-box-testing/). The dependencies of the module under test is an implementation detail that your test is not supposed to know.
* Stetting up mocks requires additional codes that makes your tests more cluttered.

So, only use mocks in a few scenarios where the mocking objects brings in more value than costs. A few cases are discussed below.

### Use Case: Mocking dependencies to simulate various test scenarios

When you try to test this system/module, sometimes you need to simulate some test scenarios where the dependencies need to behave in a controlled way. For this purpose, you need to mock the behavior of those modules/functions that your current module depend on.

For example, we have a function below that generate some random arrays. It relies on the `randomInt` function from the `mathjs` module. If you need to test different scenarios, you need to control the return values from the `randomInt` function. But how can you do that? You can mock that `randomInt` function!

```javascript
const math = require("mathjs");

const generateQueue = () => {
  const randomInteger = math.randomInt(1, 10);
  const output = Array(randomInteger).map(number => math.randomInt(-20, 50));

  return output;
};
```

### Use Case: Mocking dependencies to make test run faster and reliable

If a test case depends on a real database instance as dependency, it may fail randomly \(e.g. when the database server is not reachable for some reason\), and it may run very slowly.

To avoid those issues, we can mock the dependencies so that our test case do not depend on the status of real servers.

### Use Case: Mocking Callbacks

There are also cases when you need to pass a callback to the function you need to test, and you need to check if the callback is indeed invoked by the function. In this case, you can also pass a mock function as callback and verify it's called correctly.

## Common patterns in a test case with mocked dependencies

### Mocking dependencies with your own implementation

When you write a test case with mock objects, the test case usually follow the steps below:

1. Mock the dependencies of system under test
2. Setup the system under test \(e.g create required object instances\). Hook it up with mocked dependencies.
3. Call the API on the system under test.
4. Verify the result/behavior of the system under test
5. Verify the mocked dependencies are called as expected

![mocked dependencies](../../../.gitbook/assets/mock-objects.png)

### Monitor the interaction between components with a spy

Sometimes, you may not need to change the implementation/behavior of some function, you just want to keep an eye on it and check if it's called as expected.

![spy](../../../.gitbook/assets/spy.png)

## Creating and using mock functions

In order to mock a function, you use `jest.fn()`.

### Creating a mock

* Creating a mock: `const myMockFunction = jest.fn()`
* Creating a mock with a name: `const mockFn = jest.fn().mockName('mockedFunction')`
  * You can optionally provide a name for your mock functions, which will be displayed instead of "jest.fn\(\)" in test error output. Use this if you want to be able to quickly identify the mock function reporting an error in your test output.

### Verifying that the mock function has been called

* `expect(myMockFunction).toBeCalled()`
* Checking the specific number of times that a mock function has been called: `expect(myMockFunction).toHaveBeenCallTimes(2)`

### Verifying the arguments that were supplied to the mock

* `expect(mockFunc).toBeCalledWith(arg1, arg2)`

### Stubbing a mock function's return value

* `const myMockFunction = jest.fn(() => 42)`
* `myMockFunction.mockReturnValue(42)`
  * make `myMockFunction()` return 42 everytime you call myMockFunction\(\)
* `myMockFunction.mockReturnValueOnce('you can return anything!')` 
  * make `myMockFunction()` return this value only once \(it returns `undefined` the next time it's called\)

### Sharing the same mock across test cases

#### Clearing a mock

Each mock functions keep track of various things \(e.g. how many times it has been called\).

You'll need to clear those information beforeEach test case, so that each test case is kept independent. You can achieve this by calling `myMockFunction.mockClear()` in beforeEach.

If you find yourself calling .mockClear\(\) on multiple mocks, there is a command that let you clear all mocks in one line: `jest.clearAllMocks()`.

#### Resetting a mock

Sometimes, a mock function needs to behave differently in each test case, then you can call `myMockFunction.mockReset()` to remove the current mock return values / implementations. Then you can provide new mocked behavior.

If you find yourself calling .mockReset\(\) on multiple mocks, there is a command that let you reset all mocks in one line: `jest.resetAllMocks()`.

## Creating spies on existing functions

In order to monitor the interaction between the system under tests and its dependencies \(without mocking\), you can use [`jest.spyOn`](https://jestjs.io/docs/en/jest-object#jestspyonobject-methodname) to create a spy on the given function. After you finish testing, you can also call `mockRestore` to restore the original implementation.

In the example below, you can put a spy on `play` function in the video and check it's called with you call `playMovie` function.

Example \(in video.js\)

```javascript
const video = {
  play() {
    return true;
  },
};

module.exports = video;
```

Example \(in mediaPlayer.js\)

```javascript
const video = require("./video");

function playMovie() {
  return video.play();
}

module.exports = {
  playMovie;
}
```

Example \(in mediaPlayer.test.js\)

```javascript
const video = require('./video');
const player = require('./moviePlayer');

test('plays video', () => {
  const spy = jest.spyOn(video, 'play');
  const isPlaying = player.playMovie();

  expect(spy).toHaveBeenCalled();
  expect(isPlaying).toBe(true);

  spy.mockRestore();
});
```

## Creating and using mock modules

Besides mocking a function, you can also mock a whole JavaScript module with `jest.mock()` or `jest.doMock()`

In the example below, there are two modules:

* `someModule`, a module written by you, which exports a function
* `mathjs`, a module installed into `node_modules`, which exports a object with a function called `randomInt`
* `anotherModule` which internally calls `require("./someModule")` and `require("mathjs")`

When you write tests for `anotherModule`, you may want to mock the behavior of `someModule` so that you can simulate different test scenarios.

### Mocking a module written by you

```javascript
const myMockFunction = jest.fn(() => "dummy value");
jest.doMock("./someModule", () => {
  return myMockFunction; // Note: what you return here should match the exports in './someModule.js'
});

// Note: it's crucial that you put this next line after jest.doMock() statements
const anotherModule = require("./anotherModule.js"); 

/*
now, inside anotherModule.js, when a line says `const x = require('./someModule'), 
x is the mock function returned from the factory function inside
jest.doMock('./someModule', factoryFunction)
*/
```

### Mocking a module in node\_modules

```javascript
jest.doMock("mathjs", () => {
  return {
    randomInt: () => 42 // always return 42 when math.randomInt() is called
  }; 
});

// Note: it's crucial that you put this next line after jest.doMock() statements
const anotherModule = require("./anotherModule.js"); 

/*
// inside anotherModule.js:

const math = require('mathjs')
math.randomInt() // this will always return the stubbed value of 42
*/
```

### Difference between `jest.mock()` and `jest.doMock()`

Based on the [Jest Documentation](https://jestjs.io/docs/en/jest-object.html#jestdomockmodulename-factory-options):

> calls to jest.mock\(\) will automatically be hoisted to the top of the code block

Using the same example we gave above, you can rewrite it using `jest.mock()` and you don't need to put it before the `require` statement.

```javascript
// Note: it's crucial that you put this next line after jest.doMock() statements
const anotherModule = require("./anotherModule.js"); 

jest.mock("mathjs", () => {
  return {
    randomInt: () => 42 // always return 42 when math.randomInt() is called
  }; 
});
```

Note that here we swap the order of `mock` and `require`, and it still works.

### You can also put the mock implementations into a **mocks** directory

Besides putting the mock implementation in the test case itself, you can also put some mock implementation in a directory `__mocks__` and inform jest to load that mock implementation when `require` is called to load the module.

The benefit of this approach is that the mock implementation can be shared/reused by multiple test cases.

More details can be found in [Jest documentation on manual mocks](https://jestjs.io/docs/en/manual-mocks)

## Putting it altogether

Lab: [https://github.com/thoughtworks-jumpstart/mocks-and-stubs-lab](https://github.com/thoughtworks-jumpstart/mocks-and-stubs-lab)

Solutions: [https://github.com/songguoqiang/mocks-and-stubs-lab](https://github.com/songguoqiang/mocks-and-stubs-lab) \(don't peek unless you have to!\)

In the solutions repo, you can find examples on how to

1. Create mock functions 
   * Using `jest.fn()`
2. Make mock functions return specific values \(i.e. stubbing\)
   * Using `myMockFunction.mockReturnValue('any value')` 
   * or       `myMockFunction.mockReturnValueOnce('any value')`
3. Make expectations/assertions on mock functions
   * Using `expect(myMockFunction).toBeCalled()`
   * or       `expect(myMockFunction).toHaveBeenCalledTimes(42)`
4. Clear mocks
   * Using `myMockFunction.mockClear()`
   * or       `jest.clearAllMocks()`
5. Mock functions imported from another module \(i.e. javascript file or javascript library\)
   * Using 
   * ```text
     jest.doMock("../src/queueService.js", () => {
       return mockGenerateQueue;
     });
     ```
   * or
   * ```text
     const mockRandomInt = jest.fn()
     jest.doMock("mathjs", () => {
       return {
         randomInt: mockRandomInt
       };
     });
     ```

## Resources

### Recommended reading

* [A great explanation on how to use mocks in jest](https://medium.com/@rickhanlonii/understanding-jest-mocks-f0046c68e53c)
* [But really, what is a JavaScript mock?](https://blog.kentcdodds.com/but-really-what-is-a-javascript-mock-10d060966f7d)

### Other JavaScript libraries for mocking

* [testdouble](https://github.com/testdouble/testdouble.js)
* [sinon](https://sinonjs.org/)

### References

* [Jest documentation on mocking functions](https://jestjs.io/docs/en/mock-functions)
* [Jest documentation on manual mocks](https://jestjs.io/docs/en/manual-mocks)
* [Jest documentation on mocking ES6 classes](https://jestjs.io/docs/en/es6-class-mocks)
* [Jest Mock API reference](https://jestjs.io/docs/en/mock-function-api.html)
* [The `jest` object API](https://jestjs.io/docs/en/jest-object.html)

### Mocks, stubs, dummies, spies, whaaaat?

**Mocks** are objects/functions that can be used to \(i\) replace actual object/functions in test cases and \(ii\) verify that objects/functions are called/used \(optionally with a specific number of calls and specific arguments\)

**Stubs** provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test.

**Dummy** objects are passed around but never actually used. Usually they are just used to fill parameter lists.

**Spies** are objects which are wrapped around the actual object of interest. Spies allow us to to "spy on" actual objects and record some information on how they were called \(e.g. specific number of calls and specific arguments\)

Continue reading: [What are mocks, stubs, spies, dummies?](https://martinfowler.com/articles/mocksArentStubs.html)

