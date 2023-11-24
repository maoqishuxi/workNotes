# JEST

1. Generate a basic configuration file
   Based on your project, Jest will ask you a few questions and will create a basic configuration file with a short description for each option:

   ```
   // npm
   npm init jest@latest
   // yarn
   yarn create jest@latest
   ```

2. Using Babel

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

3. Using TypeScript

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

4. Type definitions

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

5. Using Matchers

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

6. Code Transformation

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

   

7. Truthiness

   In tests, you some

8. 

9. 

