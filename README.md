# AngularUnitTests

**Introduction to Unit Testing in Angular with Jasmine**
========================================================

Unit testing is a crucial practice for ensuring your Angular application behaves as expected. This tutorial provides a beginner-friendly introduction to unit testing Angular apps using **Jasmine** and **TestBed**. Before diving deeper into unit testing, it's important to have a basic understanding of **Dependency Injection (DI)** in Angular, as it is fundamental to configuring services and dependencies in your tests [Link here](./introduction-di-angular.md). You'll also find links to advanced topics for deeper exploration.

---

**What’s in This Guide?**
-------------------------

# Table of Contents
1. [What is Jasmine?](#what-is-jasmine)
2. [Understanding Angular Testing](#understanding-angular-testing)
3. [Setting Up TestBed](#setting-up-testbed)
4. [What is a Test? (Naming and Purpose)](#what-is-a-test)
5. [The Role of `describe` and Nesting](#the-role-of-describe-and-nesting)
6. [The `it` Function and Running Tests](#the-it-function-and-running-tests)
7. [The Arrange-Act-Assert Pattern](#the-arrange-act-assert-pattern)
8. [Shared Test Module in Angular Unit Tests](#shared-test-module-in-angular-unit-tests)
9. [Launching Browsers](#launching-browsers)
10. [Installing Required Launchers](#installing-required-launchers)
11. [Running Tests for Specific Packages in a Monorepo](#running-tests-for-specific-packages-in-a-monorepo)
12. [Testing Promises and Observables in Angular with Jasmine](#testing-promises-and-observables-in-angular-with-jasmine)
13. [Advanced Topics](#advanced-topics)
14. [Best Practices for Writing Tests](#best-practices-for-writing-tests)
15. [Summary](#summary-chapter)

---

**1. What is Jasmine?** <a id="what-is-jasmine"></a>
-----------------------

**Jasmine** is a behavior-driven development (BDD) framework for testing JavaScript. It provides tools for writing clear, expressive tests.

### Key Jasmine Features:

*   **`describe`**: Group related tests.
*   **`it`**: Define individual test cases.
*   **Matchers**: Validate outcomes (e.g., `toEqual`, `toBe`).
*   **Spies**: Mock and track dependencies.

### Example Jasmine Test:
```javascript
describe('Simple Math Tests', () => {
  it('should add numbers correctly', () => {
    const result = 1 + 1;
    expect(result).toBe(2);
  });
});
```
---

**2. Understanding Angular Testing** <a id="understanding-angular-testing"></a>
------------------------------------

Angular testing relies on **TestBed**, **Dependency Injection**, and **mocking** to simulate and verify application behavior in isolation. Testing in Angular relies on a structured environment provided by the **TestBed** utility and Angular’s **Dependency Injection (DI)** system.


### Core Concepts:

1.  **TestBed**: Configures and initializes the testing environment.
2.  **Dependency Injection**: Explicitly provide dependencies (e.g., services) to components.
3.  **Mocking**: Replace real implementations with controlled, test-specific versions.

### Testing Workflow:

1.  **Setup**:
    *   Use `TestBed` to configure a testing module, declaring all required components, imports, and providers.
2.  **Create Component/Service**:
    *   Instantiate the component or service under test using `TestBed`.
3.  **Test**:
    *   Write `describe` blocks for grouping tests and `it` blocks for individual test cases.
4.  **Assertions**:
    *   Validate the results using Jasmine’s matchers.

----

**3. Setting Up TestBed** <a id="setting-up-testbed"></a>
-------------------------

The **TestBed** is Angular’s primary tool for setting up the testing environment. You must declare all dependencies, including components, services, and imports.

```ts
import { TestBed } from '@angular/core/testing';
import { MyComponent } from './my-component';
import { MyService } from './my-service';
import { HttpClientTestingModule } from '@angular/common/http/testing';

describe('MyComponent', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [MyComponent], // Declare the component being tested
      imports: [HttpClientTestingModule], // Import required modules
      providers: [MyService], // Provide service dependencies
    });
  });

  it('should create the component', () => {
    const fixture = TestBed.createComponent(MyComponent);
    const component = fixture.componentInstance;
    expect(component).toBeTruthy(); // Assert that the component is created
  });
});
```

### Why Declare Everything?

Angular’s testing environment doesn’t automatically discover dependencies like some backend frameworks (e.g., C#). This explicit declaration ensures:
*   **Clarity**: Developers can see exactly what a component depends on.
*   **Control**: Allows overriding dependencies for testing.
*   **Isolation**: Tests remain independent of unrelated modules or services.

---

**4. What is a Test? (Naming and Purpose)** <a id="what-is-a-test"></a>
----------------------

A test verifies that a specific behavior of your application works as expected. In Jasmine:
*   Each test is an **`it` block** inside a **`describe` block**.
*   Tests are named descriptively to explain **what** the test does and **why** it’s needed.

### Naming Tests:

1.  Use **`should`** to describe the expected behavior.
    *   Example: `it('should display error when service fails')`.
    *   `it('calculates the sum of numbers correctly when inputs are valid should return 7')`
    *   `it('when the input is empty should returns null')`
2.  Keep names short but clear.
3.  Avoid overly technical language; describe behavior from a user’s perspective.

---

**5. The Role of `describe` and Nesting** <a id="the-role-of-describe-and-nesting"></a>
----------------------

The `describe` function is used to group related test cases. Nested `describe` blocks can improve clarity by organizing tests into logical subgroups.

```ts
describe('Math Utilities', () => {
  describe('Addition', () => {
    it('should add two positive numbers', () => {
      const result = 2 + 3;
      expect(result).toBe(5);
    });

    it('should add a positive and a negative number', () => {
      const result = 5 + -3;
      expect(result).toBe(2);
    });
  });

  describe('Subtraction', () => {
    it('should subtract two numbers correctly', () => {
      const result = 5 - 3;
      expect(result).toBe(2);
    });
  });
});

```

### Why Use Nested `describe` Blocks?

1.  **Improves Test Readability**: Organizes tests logically for easier navigation.
2.  **Clarifies Test Purpose**: Each `describe` block can represent a method, scenario, or functionality.
3.  **Scoped Setup**: Setup logic (`beforeEach`) can be applied at specific levels for better control.

---

**6. The `it` Function and Running Tests** <a id="the-it-function-and-running-tests"></a>
----------------------

The **`it`** function is where you define and execute an individual test case. Each `it` block should:
1.  Test one behavior or scenario.
2.  Include clear setup, execution, and assertions.

```ts
it('should fetch data successfully', () => {
  // Arrange
  const mockData = [1, 2, 3];
  spyOn(myService, 'getData').and.returnValue(of(mockData));

  // Act
  component.fetchData();

  // Assert
  expect(component.data).toEqual(mockData);
});

```


### Running Tests:

*   Jasmine automatically runs all `it` blocks defined within active `describe` blocks.
*   Skipping or focusing on tests:
    *   Use `xdescribe` or `xit` to skip tests.
    *   Use `fdescribe` or `fit` to focus on specific tests (only these will run).

```ts
xit('should fetch data successfully', () => {
  // This test will not run
});

fit('should add numbers correctly', () => {
  const result = 1 + 2;
  expect(result).toBe(3); // Only this test will run
});
```

### Best Practices for `xit` and `fit`:

1.  Avoid committing code with `xit` or `fit`—these are debugging tools, not part of final test code.
2.  Use `xit` when working on a test that isn't ready yet.
3.  Use `fit` to focus during development, but revert to regular `it` before completing the test suite.

---

**7. The Arrange-Act-Assert Pattern** <a id="the-arrange-act-assert-pattern"></a>
----------------------


The **Arrange-Act-Assert** pattern improves the readability and structure of test cases.
1.  **Arrange**: Set up the initial conditions for the test.
2.  **Act**: Execute the action being tested.
3.  **Assert**: Verify the result matches expectations.

```ts
it('should fetch data correctly', () => {
  // Arrange
  const mockData = [1, 2, 3];
  spyOn(myService, 'getData').and.returnValue(of(mockData));

  // Act
  component.fetchData();

  // Assert
  expect(component.data).toEqual(mockData);
});

```

### Assertions in Jasmine
In Jasmine, assertions are made using the `expect` function, which is a part of the Jasmine testing framework. The `expect` function provides a wide range of matchers (methods) that allow you to validate the behavior of your code by checking various conditions in your tests. Below are some of the most popular and useful assertions (matchers) in Jasmine unit tests for Angular:

#### 1. **`toBe()`**

Compares primitive values for equality using strict equality (`===`).
*   **Example:**
```ts
it('should return the correct sum', () => {
  const sum = add(2, 3);
  expect(sum).toBe(5);
});
```

#### 2. **`toEqual()`**

Used for deep equality, comparing the values of objects or arrays, unlike `toBe()` which only compares reference types.
*   **Example:**
```ts
it('should return an object with correct properties', () => {
  const user = { name: 'John', age: 30 };
  expect(user).toEqual({ name: 'John', age: 30 });
});
```

#### 3. **`toBeTruthy()` / `toBeFalsy()`**

Tests whether a value is truthy or falsy, which is useful for checking boolean conditions or verifying if a value is defined or not (undefined).
*   **Example:**
```ts
it('should return a truthy value', () => {
  const value = 'Hello';
  expect(value).toBeTruthy();  // Non-empty strings are truthy
});

it('should return a falsy value', () => {
  const value = null;
  expect(value).toBeFalsy();  // null is falsy
});
```

#### 4. **`toBeDefined()` / `toBeUndefined()`**

Checks whether a value is defined or undefined. This is useful when testing if a variable or property has been properly initialized.
*   **Example:**
```ts
it('should be defined', () => {
  const user = { name: 'Alice' };
  expect(user.name).toBeDefined();
});

it('should be undefined', () => {
  const user = {};
  expect(user.age).toBeUndefined();
});
```

#### 5. **`toContain()`**

Checks if an array or string contains a specific element or substring.
*   **Example:**
```ts
it('should contain a specific element in array', () => {
  const colors = ['red', 'green', 'blue'];
  expect(colors).toContain('green');
});

it('should contain a substring in string', () => {
  const message = 'Hello Jasmine';
  expect(message).toContain('Jasmine');
});
```

#### 6. **`toBeGreaterThan()` / `toBeLessThan()`**

Compares numerical values. Useful for checking if a number is greater or less than a given value.
*   **Example:**
```ts
it('should return a number greater than the threshold', () => {
  const result = calculateDiscount(100);
  expect(result).toBeGreaterThan(50);
});

it('should return a number less than the threshold', () => {
  const result = calculateDiscount(20);
  expect(result).toBeLessThan(30);
});
```

#### 7. **`toHaveBeenCalled()`**

Used with spies to check if a function was called.
*   **Example:**
```ts
it('should call the method when the button is clicked', () => {
  const spy = jasmine.createSpy();
  const button = new Button(spy);
  button.click();
  
  expect(spy).toHaveBeenCalled();
});
```

#### 8. **`toHaveBeenCalledWith()`**

Checks if a function was called with specific arguments.
*   **Example:**
```ts
it('should call the method with the correct arguments', () => {
  const spy = jasmine.createSpy();
  const button = new Button(spy);
  button.click(10, 'click');

  expect(spy).toHaveBeenCalledWith(10, 'click');
});
```

#### 9. **`toThrow()`**

Checks if a function throws an error.
*   **Example:**
```ts
it('should throw an error if the value is negative', () => {
  const negativeValue = () => { return checkValue(-1); };
  expect(negativeValue).toThrow(new Error('Value cannot be negative'));
});
```

#### 10. **`toBeInstanceOf()`**

Checks if an object is an instance of a given class or constructor function. This is particularly useful when verifying the type of objects returned from services or factories.
*   **Example:**
```ts
it('should be an instance of the class', () => {
  const user = new User('Alice');
  expect(user).toBeInstanceOf(User);
});
```

#### 11. **`toBeNull()`**

Verifies that a value is `null`.
*   **Example:**
```ts
it('should return null if the item is not found', () => {
  const item = findItem('non-existent-id');
  expect(item).toBeNull();
});
```

#### 12. **`toMatch()`**

Checks if a string matches a regular expression.
*   **Example:**
```ts
it('should match the email format', () => {
  const email = 'user@example.com';
  expect(email).toMatch(/^[\w-]+(\.[\w-]+)*@([\w-]+\.)+[a-zA-Z]{2,7}$/);
});
```

#### 13. **`toBeNaN()`**

Verifies if the value is NaN (Not-a-Number). This is useful for checking invalid mathematical results.
*   **Example:**
```ts
it('should return NaN when dividing by zero', () => {
  const result = divide(1, 0);
  expect(result).toBeNaN();
});
```

#### 14. **`toEqual()`**

Checks if objects or arrays are deeply equal (not just by reference). This is particularly useful when comparing complex objects like arrays, dates, or instances of classes.
*   **Example:**
```ts
it('should return an array with the correct values', () => {
  const expected = [1, 2, 3];
  const result = [1, 2, 3];
  expect(result).toEqual(expected); // deep equality check
});
```

#### 15. **`toBeCloseTo()`**

Checks if a floating-point number is close to another value within a given number of decimal places. This is helpful when dealing with calculations involving floating-point precision.
*   **Example:**
```ts
it('should return a value close to the expected value', () => {
  const result = 0.1 + 0.2;
  expect(result).toBeCloseTo(0.3, 5);  // checks to 5 decimal places
});
```

#### 16. **`toBeDefined()` / `toBeUndefined()`**

Useful for checking if an object property or variable is defined or not.
*   **Example:**
```ts
it('should be defined', () => {
  const user = { name: 'Alice' };
  expect(user.name).toBeDefined();
});

it('should be undefined', () => {
  const user = {};
  expect(user.age).toBeUndefined();
});
```
### Summary of Common Assertions in Jasmine

*   **Equality Comparisons:**
    *   `toBe()`
    *   `toEqual()`
    *   `toBeCloseTo()`
*   **Truthiness/Existence:**
    *   `toBeTruthy()`
    *   `toBeFalsy()`
    *   `toBeDefined()`
    *   `toBeUndefined()`
*   **Array and String Checks:**
    *   `toContain()`
    *   `toMatch()`
*   **Function Spies:**
    *   `toHaveBeenCalled()`
    *   `toHaveBeenCalledWith()`
*   **Error and Exception Checks:**
    *   `toThrow()`
    *   `toBeNaN()`
*   **Type/Instance Checks:**
    *   `toBeInstanceOf()`
    *   `toBeNull()`
---

**8. Shared Test Module in Angular Unit Tests** <a id="shared-test-module-in-angular-unit-tests"></a>
----------------------

In large Angular applications, it’s common to have repeated setup logic across multiple test files, such as importing shared modules, mocking providers, or configuring dependencies. A **Shared Test Module** consolidates these repetitive configurations into a reusable module, improving maintainability and reducing boilerplate code.

### What is a Shared Test Module?

A **Shared Test Module** is a dedicated Angular module defined specifically for tests. It includes common test configurations, such as:
*   **Imports**: Frequently used Angular modules (e.g., `HttpClientTestingModule`, `ReactiveFormsModule`).
*   **Providers**: Shared mock services, custom tokens, or default dependencies.
*   **Declarations**: Common components, directives, or pipes needed for testing.

### When to Use a Shared Test Module

1.  **Reusability**: When multiple test files require the same imports, providers, or declarations.
2.  **Simplification**: To reduce boilerplate code in `TestBed.configureTestingModule` setups.
3.  **Consistency**: To ensure a uniform configuration across tests and avoid duplication errors.
4.  **Ease of Updates**: Changes to shared dependencies (e.g., mock services or modules) can be made in a single place.

### How to Create a Shared Test Module

#### Approach 1: Using a `SharedTestModule`
##### Example: Setting Up a Shared Test Module

```ts
import { NgModule } from '@angular/core';
import { HttpClientTestingModule } from '@angular/common/http/testing';
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [
    HttpClientTestingModule, // Mock HTTP module for tests
    ReactiveFormsModule, // Needed for forms
  ],
  providers: [
    { provide: 'API_URL', useValue: 'http://mock-api.com' }, // Custom token
    { provide: SomeService, useClass: MockSomeService }, // Mocked service
  ],
})
export class SharedTestModule {}
```

#### Using the Shared Test Module in Tests

Import the **SharedTestModule** in your `TestBed` setup:
```ts
import { TestBed } from '@angular/core/testing';
import { MyComponent } from './my.component';
import { SharedTestModule } from './shared-test.module';
import { AnotherService } from './another.service';

describe('MyComponent', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [SharedTestModule], // Import the shared test module
      declarations: [MyComponent],
      providers: [
        { provide: AnotherService, useValue: { getValue: () => 'Mock Value' } }, // Override
      ],
    }).compileComponents();
  });

  it('should use overridden AnotherService', () => {
    const fixture = TestBed.createComponent(MyComponent);
    const component = fixture.componentInstance;
    const service = TestBed.inject(AnotherService);

    expect(service.getValue()).toBe('Mock Value'); // Confirm mock service is used
  });
});
```

#### Approach 2: Using `useSharedTestingModule` function
##### Utility Function

Define a utility function that configures common test dependencies dynamically.
```ts
// shared-testing-utils.ts
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule } from '@angular/common/http/testing';
import { SomeService } from './some.service';
import { MockSomeService } from './mock-some.service';

export function useSharedTestingModule(): void {
  TestBed.configureTestingModule({
    imports: [HttpClientTestingModule],
    providers: [
      { provide: SomeService, useClass: MockSomeService }, // Default mock
    ],
  });
}

```
##### Test Case Using Utility Function

Override dependencies dynamically for the test.

```ts
import { TestBed } from '@angular/core/testing';
import { useSharedTestingModule } from './shared-testing-utils';
import { MyComponent } from './my.component';
import { AnotherService } from './another.service';

describe('MyComponent', () => {
  beforeEach(() => {
    // Use shared testing setup
    useSharedTestingModule();

    // Add or override additional configurations
    TestBed.configureTestingModule({
      declarations: [MyComponent],
      providers: [
        { provide: AnotherService, useValue: { getValue: () => 'Overridden Mock Value' } },
      ],
    }).compileComponents();
  });

  it('should use overridden AnotherService', () => {
    const fixture = TestBed.createComponent(MyComponent);
    const component = fixture.componentInstance;
    const service = TestBed.inject(AnotherService);

    expect(service.getValue()).toBe('Overridden Mock Value'); // Confirm overridden mock service is used
  });
});

```
#### Key Differences and Notes

| **Aspect** | **SharedTestModule** | **useSharedTestingModule** |
| --- | --- | --- |
| **Centralized Setup** | All common dependencies are defined in one place. | Common setup is dynamic and flexible per test. |
| **Ease of Use** | Easy to use with minimal boilerplate. | Requires explicit TestBed configuration. |
| **Dependency Overrides** | Done directly in test files as needed. | Overrides are more explicit and modular. |
| **Flexibility** | Limited to what’s included in the module. | High flexibility to tailor per test. |



### Benefits of a Shared Test Module

1.  **Improved Test Readability**: Reduces clutter in test files by abstracting common configurations.
2.  **Centralized Configuration**: Simplifies updates and ensures consistent testing environments.
3.  **Scalability**: As the application grows, adding new shared dependencies becomes easier.

### When _Not_ to Use a Shared Test Module

While Shared Test Modules are useful, they may not suit every scenario:
1.  **Highly Specific Test Configurations**: If tests require unique dependencies or setups.
2.  **Overhead in Small Projects**: For small apps or test suites, the added abstraction may be unnecessary.
3.  **Test Isolation Concerns**: Overusing shared dependencies might lead to unintentional coupling between tests.

---

**9. Launching Browsers** <a id="launching-browsers"></a>
----------------------

In Angular projects using **Karma** as the test runner, you can configure which browser(s) to use for running your unit tests in the `karma.conf.js` file. This configuration file contains a section for defining the browsers, and you can also install plugins for specific browsers if needed.
Here’s how you can configure it:

### 1. **Locate `karma.conf.js`**

The file is usually located in the root directory of your Angular project.

### 2. **Browser Configuration**

The `browsers` property in the `karma.conf.js` file defines the browsers to use.
Example:
```js
module.exports = function (config) {
  config.set({
    // Other configurations...
    
    browsers: ['Chrome'], // Default browser

    // Plugins must be included if you're using non-default browsers
    plugins: [
      'karma-chrome-launcher',
      'karma-firefox-launcher'
    ],
  });
};
```

### 3. **Common Browsers and Launchers**

| Browser | Launcher Plugin | Description |
| --- | --- | --- |
| **Chrome** | `karma-chrome-launcher` | Runs tests in Google Chrome. |
| **Firefox** | `karma-firefox-launcher` | Runs tests in Mozilla Firefox. |
| **Edge** | `karma-edge-launcher` | Runs tests in Microsoft Edge. |
| **Safari** | `karma-safari-launcher` | Runs tests in Safari (macOS only). |
| **PhantomJS** | `karma-phantomjs-launcher` | Runs tests in a headless browser. (deprecated) |

### 4. **Switching Between Browsers**

Modify the `browsers` property to use a different browser. For example:

```javascript
browsers: ['Firefox']
```

You can also specify multiple browsers:

```javascript
browsers: ['Chrome', 'Firefox']
```

### 5. **Using Headless Browsers**

Headless browsers are useful for CI pipelines where no GUI is available. For example:

```javascript
browsers: ['ChromeHeadless']
```

Ensure the launcher is installed:
```bash
npm install --save-dev karma-chrome-launcher
```

### 6. **Running Specific Browser from CLI**

You can override the browser defined in `karma.conf.js` using the command line:
```bash
npm run test -- --browsers=Chrome
npm run test -- --browsers=Firefox
```

---

**10. Installing Required Launchers** <a id="installing-required-launchers"></a>
----------------------

Make sure you install the corresponding browser launcher for the browser you want to use. For example:

```bash
npm install --save-dev karma-chrome-launcher karma-firefox-launcher
```


Example of `karma.conf.js` with Browser Configuration:
```js
module.exports = function (config) {
  config.set({
    frameworks: ['jasmine', '@angular-devkit/build-angular'],
    plugins: [
      require('karma-jasmine'),
      require('karma-chrome-launcher'),
      require('karma-firefox-launcher'),
      require('@angular-devkit/build-angular/plugins/karma'),
    ],
    client: {
      clearContext: false, // leave Jasmine Spec Runner output visible in browser
    },
    reporters: ['progress'],
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: true,
    browsers: ['Chrome'], // Set default browser
    singleRun: false,
    restartOnFileChange: true,
  });
};
```


### Summary

*   **`browsers`**: Specifies the browser(s) for tests.
*   **Install necessary launchers**: Ensure you have plugins like `karma-chrome-launcher`.
*   **Command-line override**: Use `npm run test -- --browsers=<browser>` to run in a specific browser.

---

**11. Running Tests for Specific Packages in a Monorepo** <a id="running-tests-for-specific-packages-in-a-monorepo"></a>
----------------------

In a monorepo, where multiple packages or workspaces are maintained within a single repository, there might be situations where you want to run tests for just one package rather than the entire project. This is particularly useful to speed up the development process and avoid running unnecessary tests in unrelated packages.
To run tests for a specific package, you can configure your `npm` commands and scripts to handle arguments that specify the target package. Here’s how you can set it up:

### Using **npm workspaces** or a **custom script**:

If your monorepo supports passing arguments (e.g., **npm workspaces** or using a custom script), you can execute tests for a specific package as follows:

```bash
npm run test -- <package-name>
```

This command will run the tests only for the specified package. However, for this to work, your `test` script in the `package.json` should be set up to handle such arguments.

### Example `package.json` Setup:
```javascript
{
  "name": "demo",
  "scripts": {
    "ng": "ng",
    "start": "npm run start",
    "build": "ng build",
    "build:package": "ng build package",
    "test": "ng test",
    "test:silent": "ng test --watch=false --code-coverage=true"
  },
  // ... other setup
}
```

In this configuration, when you use `npm run test`, it will trigger `ng test`, which runs tests for the entire monorepo. However, if you want to run tests for a specific package, you can pass that package as an argument to `npm run test` (e.g., `npm run test -- <package-name>`).

For a CI/CD pipeline, it’s important to run unit tests in **silent mode** by configuring `--watch=false` and `--single-run=true` to ensure tests are executed only once and do not require any user interaction. This way, tests can run efficiently in the pipeline, generate results, and exit automatically

---

**12. Testing Promises and Observables in Angular with Jasmine** <a id="testing-promises-and-observables-in-angular-with-jasmine"></a>
----------------------

This section would cover how to test asynchronous code, particularly Promises and Observables, which are commonly used in Angular applications.

### Description

In Angular unit tests, handling asynchronous code—such as Promises and Observables—requires special care. Jasmine provides tools like `done: DoneFn` and `expectAsync` to deal with such code. Here's a breakdown of how to handle them effectively.

#### Testing Promises

When testing code that returns a Promise, Jasmine requires you to ensure that the test waits for the promise to resolve (or reject). Here’s how you can structure a test for a Promise:

```ts
it('should resolve the promise correctly', async () => {
  await someAsyncFunction();
  expect(response).toBe('Expected value');
});
```

In this example:
*   `done` is called after the promise resolves to signal the test has finished.
*   If the promise rejects, the test will fail using `done.fail()`.

#### Testing Observables

When testing Observables, you can subscribe to the Observable and use Jasmine’s `done` function or `async` utilities. It's important to ensure you test the values emitted by the Observable properly. For Observables, we often use `expectAsync` to wrap asynchronous code.

```ts
it('should emit values from observable', (done: DoneFn) => {
  someObservable().subscribe({
    next: (response) => {
      expect(response).toBe('Expected value');
      done();
    },
    error: (error) => {
      done.fail('Observable failed');
    }
  });
});
```

#### Mixing `done: DoneFn` with `inject()`

In some cases, you may need to combine `done: DoneFn` with `inject()` to inject dependencies and ensure that asynchronous code is properly tested.
Example:

```ts
it('should call the service and resolve the observable', (done: DoneFn) => {
  inject([MyService], (myService: MyService) => {
    myService.getData().subscribe({
      next: (response) => {
        expect(response).toEqual('Expected data');
        done();
      },
      error: (err) => {
        done.fail('Observable failed');
      }
    });
  })();
});
```

This approach ensures that the `inject()` function is used to inject dependencies, while `done()` signals the completion of asynchronous code.

#### Using `await` and `expectAsync` for Promises

When testing asynchronous functions, you can use `await` combined with `expectAsync` to simplify the testing process, especially with promises that may reject. `expectAsync` is necessary to properly handle Promises and ensures that assertions are made after the promise resolves or rejects.
Example with `expectAsync`:

```ts
it('should reject the promise with an error', async () => {
  await expectAsync(someAsyncFunction()).toBeRejectedWithError('Expected error');
});
```

For promises, **always use `async` and `await`** to ensure that the test waits for the promise resolution. If you don't use `async`, Jasmine might complete the test before the Promise resolves, causing unreliable results.

### Key Takeaways:

1.  **Always use `done: DoneFn`** for asynchronous operations with Observables in Jasmine. This ensures the test waits for completion.
2.  **Use `expectAsync`** for promises and async functions to handle resolutions or rejections properly.
3.  **Always use `async` and `await`** when dealing with promises to ensure your test waits for the result before completing.

---

**13. Advanced Topics** <a id="advanced-topics"></a>
----------------------

Explore these advanced testing techniques in separate guides:
1.  **Wrapping Global Objects**:
    *   Learn how to create and test wrapper services for `window` or `document`.
    *   ➡️ See [Wrapping Global Objects]([Modifying Component Metadata - Overview]([Wrapping Global Objects - Overview](link))) for more information on how to create and test these wrappers.
2.  **Using Spies in Tests**:
    *   Discover advanced spy techniques (`spyOn`, `createSpyObj`) and when to use them.
    * ➡️ See [Using Spies in Tests]([Using Spies in Tests - Overview](link)) for advanced techniques and examples.
3.  **Recreating and Overriding Dependencies**:
    *   Understand how to override services or inject mocks into tests.
    * ➡️ See [Recreating and Overriding Dependencies]([Recreating and Overriding Dependencies - Overview](link)) for detailed examples and use cases.
4.  **Modifying Component Metadata**:
    *   Simplify testing by overriding component templates or metadata.
    * ➡️ See [Modifying Component Metadata]([Modifying Component Metadata - Overview](link)) for more details.

---

**14. Best Practices for Writing Tests** <a id="best-practices-for-writing-tests"></a>
----------------------

1.  **Focus on One Thing**:
    *   Each test should verify a single behavior or functionality.
2.  **Mock External Dependencies**:
    *   Avoid real HTTP calls or database interactions; use mock data or spies.
3.  **Name Tests Clearly**:
    *   Use descriptive names that include the word "should" for clarity.
    *   Example: `it('should display error when service fails')`.
4.  **Organize Tests**:
    *   Use `describe` to group related tests and nested `describe` for subgroups.
    *  Name Tests Clearly**:
    *   Include "should" in test names to describe behavior.
    *   Example: `it('should return empty array when no data is found')`.
    *   Avoid duplicating the `describe` name in `it` names. For example

---

**15. Summary** <a id="summary-chapter"></a>
----------------------

### Key Concepts:

1.  Use `describe` to group related tests and organize them logically.
2.  Write clear, focused test names with the word "should."
3.  Use nested `describe` blocks to test methods, scenarios, or behaviors independently.

### Benefits of These Practices:

*   **Readability**: Structured tests make it easy for developers to understand the functionality being tested.
*   **Maintainability**: Logical organization ensures tests are easier to update as the application evolves.
*   **Clarity**: Naming conventions and grouping make test intent immediately clear.

