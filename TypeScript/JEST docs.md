# JEST

1. nextjs add jest library usage (base: react, typescript):

   - To set up Jest, install `jest`, `jest-environment-jsdom`, `@testing-library/react`, `@testing-library/jest-dom`, `@types/jest`:

     ```
     npm install --save-dev jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom @types/jest
     ```

   - Create a `jest.config.mjs` file in your project's root directory and add the following:

     ```
     // jest.config.mjs
     
     import nextJest from 'next/jest.js'
      
     const createJestConfig = nextJest({
       // Provide the path to your Next.js app to load next.config.js and .env files in your test environment
       dir: './',
     })
      
     // Add any custom config to be passed to Jest
     /** @type {import('jest').Config} */
     const config = {
       // Add more setup options before each test is run
       
       setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
       testEnvironment: 'jest-environment-jsdom',
     }
      
     // createJestConfig is exported this way to ensure that next/jest can load the Next.js config which is async
     export default createJestConfig(config)
     ```

     **Optional: Extend Jest with custom matchers**

     `@testing-library/jest-dom` includes a set of convenient [custom matchers](https://github.com/testing-library/jest-dom#custom-matchers) such as `.toBeInTheDocument()` making it easier to write tests. You can import the custom matchers for every test by adding the following option to the Jest configuration file:

     ```
     // jest.config.js
     
     setupFilesAfterEnv: ['<rootDir>/jest.setup.js']
     ```

   - Then, inside `jest.setup.js`, add the following import:

     ```
     // jest.setup.js
     
     import '@testing-library/jest-dom'
     ```

   - Add the Jest executable in watch mode to the `package.json` scripts:

     ```
     // package.json
     
     {
       "scripts": {
         "dev": "next dev",
         "build": "next build",
         "start": "next start",
         "test": "jest --watch"
       }
     }
     ```

   - If you don't use Babel it's enough to add `@testing-library/jest-dom` to your types. and then link it in your *tsconfig.json*:

     ```
     "types": ["node", "jest", "@testing-library/jest-dom"],
     ```

2. Generate a basic configuration file
   Based on your project, Jest will ask you a few questions and will create a basic configuration file with a short description for each option:

   ```
   // npm
   npm init jest@latest
   // yarn
   yarn create jest@latest
   ```

3. Using Babel

   To use Babel, install required dependencies:

   ```
   npm install --save-dev babel-jest @babel/core @babel/preset-env
   ```

   Configure Babel to target your current version of Node by creating a `babel.config.js` file in the root of your project:

   ```
   module.exports = {
     presets: [['@babel/preset-env', {targets: {node: 'current'}}]],
   };
   ```

4. Using TypeScript

   Via `babel`

   Jest supports TypeScript, via Babel. First, make sure you followed the instructions on using Babel above. Next, install the `@babel/preset-typescript`:

   ```
   // npm
   npm install --save-dev @babel/preset-typescript
   // yarn
   yarn add --dev @babel/preset-typescript
   ```

   Then add `@babel/preset-typescript` to the list of presets in your `babel.config.js`.

   ```
   module.exports = {
     presets: [
       ['@babel/preset-env', {targets: {node: 'current'}}],
       '@babel/preset-typescript',
     ],
   };
   ```

   Because TypeScript support in Babel is purely transpilation, Jest will not type-check your tests as they are run. If you want that, you can use `ts-jest` instead, or just run the TypeScript compiler `tsc` separately (or as part of your build process).

   Via `ts-jest`

   `ts-jest` is a TypeScript preprocessor with source map support for Jest that lets you use Jest to test projects written in TypeScript.

   ```
   npm install --save-dev ts-jest
   ```

   In order for Jest to transpile TypeScript with `ts-jest`, you may also need to create a `configuration` file.

   ```
   npx ts-jest config:init
   ```

5. Type definitions

   There are two ways to have `Jest global APIs` typed for test files written in TypeScript.

   You can use type type definitions which ships with Jest and will update each time you update Jest. Install the `@jest/globals` packages:

   ```
   // npm
   npm install --save-dev @jest/globals
   // yarn
   yarn add --dev @jest/globals
   ```

   And import the APIs from it:

   ```
   // sum.test.ts
   import {describe, expect, test} from '@jest/globals';
   import {sum} from './sum';
   
   describe('sum module', () => {
     test('adds 1 + 2 to equal 3', () => {
       expect(sum(1, 2)).toBe(3);
     });
   });
   ```

   Or you may choose to install the `@types/jest` package. It provides types for Jest globals without a need to import them.

   ```
   // npm
   npm install --save-dev @types/jest
   // yarn
   yarn add --dev @types/jest
   ```

6. Using Matchers

   Jest uses "matchers" to let you test values in different ways.

   Common Matchers

   The simplest way to test a value is with exact equality.

   ```
   test('two plus two is four', () => {
   	expect(2 + 2).toBe(4);
   });
   ```

   `toBe` uses `Object.is` to test exact equality. If you want to check the value of an object, use `toEqual`:

   ```
   test('object assignment', () => {
   	const data = {one: 1};
   	data['two'] = 2;
   	expect(data).toEqual({one: 1, two: 2});
   });
   ```

   You can also test for the opposite of a matcher using `not`:

   ```
   test('adding positive numbers is not zero', () => {
   	for (let a = 1; a < 10; a++) {
   		for (let b = 1; b < 10; b++) {
   			expect(a + b).not.toBe(0);
   		}
   	}
   });
   ```

7. Code Transformation

   Jest runs the code in your project as JavaScript, but if you use some syntax not supported by Node out of the box (such as JSX, TypeScript, Vue templates) then you'll need to transform that code into plain JavaScript, similar to what you would do when building for browsers.

   Jest support this via the `transform` configuration option.

   ```
   // add custom-sequencer to your Jest configuration
   import type {Config} from 'Jest';
   
   const config: Config = {
   	testSequencer: 'path/to/custom-sequencer.js',
   };
   
   export default config;
   ```

   Defaults

   Jest ships with one transformer out of the box -- `babel-jest`. It will load your project's Babel configuration and transform any file matching the `/\.[jt]sx?$/` RegExp (in other words, any `.js`, `.jsx`, `.ts` or `.tsx` file). In addition, `babel-jest` will inject the Babel plugin necessary for mock hoisting talked about in `ES Module mocking`.

   Remember to include the default `babel-jest` transformer explicitly, if you wish to use it alongside with additional code preprocessors:

   ```
   "transform": {
     "\\.[jt]sx?$": "babel-jest",
     "\\.css$": "some-css-transformer",
   }
   ```

8. Truthiness

   In tests, you sometimes need to distinguish between `undefined`, `null`, and `false`, but you sometimes do not want to treat these differently. Jest contains helpers that let you be explicit about what you want.

   - `toBeNull` matches only `null`
   - `toBeUndefined` matches only `undefined`
   - `toBeDefined` is the opposite of `toBeUndefined`
   - `toBeTruthy` mathes anything that an `if` statement treats as true
   - `toBeFalsy` matches anything that an `if` statement treats as false

9. Testing Asynchronous Code

   It's common in JavaScript for code to run asynchronously. When you have code that runs asynchronously, Jest needs to know when the code it is testing has completed, before it can move on to another test.

10. Promises

    Return a promise from your test, and Jest will wait for that promise to resolve. If the promise is rejected, the test will fail.

    ```
    test('the data is peanut butter', () => {
    	return fetchData().then(data => {
    		expect(data).toBe('peanut butter');
    	});
    });
    ```

11. Async/Await

    Alternatively, you can use `async` and `await` in your tests. To write an async test, use the `async` keyword in front of the function passed to `test`.

    ```
    test('the data is peanut butter', async () => {
    	const data = await fetchData();
    	expect(data).toBe('peanut butter');
    });
    
    test('the fetch fails with an error', async () => [
    	expect.assertions(1);
    	try {
    		await fetchData();
    	} catch (e) {
    		expect(e).toMatch('error');
    	}
    ]);
    ```

    You can combine `async` and `await` with `.resolves` or `.rejects`.

    ```
    test('the data is peanut butter', async () => {
    	await expect(fetchData()).resolves.toBe('peanut butter');
    });
    
    test('the fetch fails with an error', async () => {
    	await expect(fetchData()).rejects.toMatch('error');
    });
    ```

12. Setup and Teardown

    Often while writing tests you have some setup work that needs to happen before tests run, and you have some finishing work that needs to happen after tests run. Jest provides helper functions to handle this.

13. Repeating Setup

    If you have some work you need to do repeatedly for many tests, you can use `beforeEach` and `afterEach` hooks.

    ```
    beforeEach(() => {
    	initializeCityDatabase();
    })
    
    afterEach(() => {
    	clearCityDatabase();
    });
    
    test('city database has Vienna', () => {
    	expect(isCity('Vienna')).toBeTruthy();
    });
    
    test('city database has San Juan', () => {
    	expect(isCity('San Juan')).toBeTruthy();
    });
    ```

14. Scoping

    The top level `before*` and `after*` hooks apply to every test in a file. The hooks declared inside a `describe` block apply only to the tests within that `describe` block.

    ```
    // Applies to all tests in this file
    beforeEach(() => {
      return initializeCityDatabase();
    });
    
    test('city database has Vienna', () => {
      expect(isCity('Vienna')).toBeTruthy();
    });
    
    test('city database has San Juan', () => {
      expect(isCity('San Juan')).toBeTruthy();
    });
    
    describe('matching cities to foods', () => {
      // Applies only to tests in this describe block
      beforeEach(() => {
        return initializeFoodDatabase();
      });
    
      test('Vienna <3 veal', () => {
        expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
      });
    
      test('San Juan <3 plantains', () => {
        expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
      });
    });
    ```

15. Order of Execution

    Jest executes all describe handlers in a test file *before* it executes any of the actual tests. This is another reason to do setup and teardown inside `before*` and `after*` handlers rather than inside the `describe` blocks. Once the `describe` blocks are complete, by default Jest runs all the tests serially in the order they were encountered in the collection phase, waitting for each to finish and be tidied up before moving on.

    ```
    describe('describe outer', () => {
      console.log('describe outer-a');
    
      describe('describe inner 1', () => {
        console.log('describe inner 1');
    
        test('test 1', () => console.log('test 1'));
      });
    
      console.log('describe outer-b');
    
      test('test 2', () => console.log('test 2'));
    
      describe('describe inner 2', () => {
        console.log('describe inner 2');
    
        test('test 3', () => console.log('test 3'));
      });
    
      console.log('describe outer-c');
    });
    
    // describe outer-a
    // describe inner 1
    // describe outer-b
    // describe inner 2
    // describe outer-c
    // test 1
    // test 2
    // test 3
    ```

16. General Advice

    If a test is failing, one of the first things to check should be whether the test is failing when it's the only test that runs. To run only one test with Jest, temporarily change that `test` command to a `test.only`:

    ```
    test.only('this will be the only test that runs', () => {
    	expect(true).toBe(false);
    });
    
    test('this test will not run', () => {
    	expect('A').toBe('A');
    });
    ```

17. Mock Functions

    Mock functions allow you to test the links between code by erasing the actual implementation of a function, capturing calls to the function (and the parameters passed in those calls), capturing instances of constructor functions when instantiated with `new`, and allowing test-time configuration of return values.

    There are two ways to mock functions: Either by creating a mock function to use in test code, or writing a `manual mock` to override a module dependency.

18. Using a mock function

    Let's imagine we're testing an implementation of a function `forEach`, which invokes a callback for each item in a supplied array.

    ```
    // forEach.js
    
    export function forEach(items, callback) {
      for (let index = 0; index < items.length; index++) {
        callback(items[index]);
      }
    }
    ```

    To test this function, we can use a mock function, and inspect the mock's state to ensure the callback is invoked as expected.

    ```
    // forEach.test.js
    
    const forEach = require('./forEach');
    
    const mockCallback = jest.fn(x => 42 + x);
    
    test('forEach mock function', () => {
    	forEach([0, 1], mockCallback);
    	
    	// The mock function was called twice
      expect(mockCallback.mock.calls).toHaveLength(2);
    
      // The first argument of the first call to the function was 0
      expect(mockCallback.mock.calls[0][0]).toBe(0);
    
      // The first argument of the second call to the function was 1
      expect(mockCallback.mock.calls[1][0]).toBe(1);
    
      // The return value of the first call to the function was 42
      expect(mockCallback.mock.results[0].value).toBe(42);
    });
    ```

19. Snapshot Testing

    A typical snapshot test case renders a UI component, takes a snapshot, then compares it to a reference snapshot file stored alongside the test. The test will fail if the two snapshots do not match: either the change is unexpected, or the reference snapshot needs to be updated to the new version of the UI component.

20. Snapshot Testing with Jest

    A similar approach can be taken when it comes to testing your React components. Instead of rendering the graphical UI, which would require building the entire app, you can use a test renderer to quickly generate a serializable value for your React tree. Consider this example test for a Link component:

    ```
    import renderer from 'react-test-renderer';
    import Link from '../Link';
    
    it('renders correctly', () => {
      const tree = renderer
        .create(<Link page="http://www.facebook.com">Facebook</Link>)
        .toJSON();
      expect(tree).toMatchSnapshot();
    });
    ```

21. Updating Snapshots

    It's straightforward to spot when a snapshot test fails after a bug has been introduced. When that happens, go ahead and fix the issue and make sure your snapshot tests are passing again. Now, let's talk about the case when a snapshot test is failing due to an intentional implementation change.

    ```
    // Updated test case with a Link to a different address
    it('renders correctly', () => {
      const tree = renderer
        .create(<Link page="http://www.instagram.com">Instagram</Link>)
        .toJSON();
      expect(tree).toMatchSnapshot();
    });
    ```

