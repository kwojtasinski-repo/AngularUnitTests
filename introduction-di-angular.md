# Introduction

Dependency Injection (DI) is a design pattern that Angular extensively uses to provide dependencies (services, objects, or values) to components and other parts of your application. Understanding DI is crucial for writing effective and isolated unit tests in Angular.

- [Introduction](#introduction)
	- [**What is Dependency Injection in Angular?**](#what-is-dependency-injection-in-angular)
		- [**How Angular Handles DI**](#how-angular-handles-di)
			- [**Example of DI in Angular**](#example-of-di-in-angular)
	- [**Injection via Constructors vs. Functions (`inject`)**](#injection-via-constructors-vs-functions-inject)
	- [**Tokens in Dependency Injection**](#tokens-in-dependency-injection)
	- [**How Providers Work**](#how-providers-work)
		- [**Types of Providers**](#types-of-providers)
		- [**How Providers Work Outside Unit Tests**](#how-providers-work-outside-unit-tests)
	- [**Dependency Injection (DI) in Unit Testing**](#dependency-injection-di-in-unit-testing)
	- [**Inject vs. TestBed.inject**](#inject-vs-testbedinject)
	- [**Setting Up DI in Tests**](#setting-up-di-in-tests)
		- [**Example: Setting Up DI for a Component Test**](#example-setting-up-di-for-a-component-test)
				- [**Original Implementation**](#original-implementation)
			- [**Testing the Component with DI**](#testing-the-component-with-di)
	- [**Modern Usage with the `inject` Function**](#modern-usage-with-the-inject-function)
			- [Example of `inject` in Tests:](#example-of-inject-in-tests)
	- [**Conclusion**](#conclusion)
		- [**Key Takeaways**](#key-takeaways)



* * *

## **What is Dependency Injection in Angular?**

Dependency Injection allows you to inject required services or objects into a class rather than creating them inside the class itself. This promotes better modularity, testability, and maintainability.

### **How Angular Handles DI**

*   Angular uses a **dependency injection system** where services and other dependencies are registered and provided to any class that declares them as dependencies.
*   You can declare dependencies in:
    *   **Constructors:** Commonly used in components, services, and directives.
    *   **Functions (using `inject()`):** Available primarily in tests or Angular features like standalone APIs.

In Angular:
1.  **Services** are registered in a **dependency injection system**, typically by using the `@Injectable()` decorator. This decorator makes the service available for injection across the application.
2.  Components, directives, pipes, and other services declare their dependencies via the constructor, which Angular inspects to determine what to inject.

* * *

#### **Example of DI in Angular**
```ts
@Injectable()
export class DataService {
  getData() {
    return ['item1', 'item2'];
  }
}

@Component({
  selector: 'app-my-component',
  template: '<div></div>',
})
export class MyComponent {
  constructor(private dataService: DataService) {}

  fetchData() {
    return this.dataService.getData();
  }
}

```

Here, the `DataService` is injected into the `MyComponent` class via its constructor.



* * *

## **Injection via Constructors vs. Functions (`inject`)**

1.  **Constructor-Based Injection**
    *   Most common in Angular applications.
    *   Dependencies are injected by specifying them in the class constructor.
    *   Simplifies usage and keeps the dependency lifecycle tied to the parent class.

Example:

```ts
@Injectable()
export class DataService {
  getData() {
    return ['item1', 'item2'];
  }
}

@Component({
  selector: 'app-my-component',
  template: '<div></div>',
})
export class MyComponent {
  constructor(private dataService: DataService) {}

  fetchData() {
    return this.dataService.getData();
  }
}

```

**Function-Based Injection (`inject`)**
*   Used primarily in tests and Angular's standalone features.
*   Allows direct retrieval of services within a function's scope without needing to define a class or constructor.

Usage of `inject()`
✅ simplifies inheritance
✅ enables functional provider
❌ only available in injection context

Example in Tests:

```ts
import { inject } from '@angular/core/testing';

it('should fetch data', inject([DataService], (dataService: DataService) => {
  spyOn(dataService, 'getData').and.returnValue(['mockData']);
  expect(dataService.getData()).toEqual(['mockData']);
}));

```

Example in Standalone Components:
```ts
import { inject } from '@angular/core';

const service = inject(DataService);
service.getData();

```

* * *

## **Tokens in Dependency Injection**

While services and classes are common dependencies, Angular provides a mechanism called **tokens** for more flexibility. Tokens are especially useful when you need to inject values, interfaces, or abstract classes.
1.  **What Are Tokens?** Tokens act as unique identifiers for dependencies, allowing Angular to differentiate between services or provide values that don’t map directly to a class. They are created using `InjectionToken`.

```ts
import { InjectionToken } from '@angular/core';

export const API_URL = new InjectionToken<string>('API_URL');
```

2.  **Using Tokens for Interfaces** Since interfaces do not exist at runtime, they cannot be directly injected. Tokens bridge this gap by associating interfaces with runtime values or implementations.

Example with an Interface:

```ts
export interface Logger {
  log(message: string): void;
}

export const LOGGER_TOKEN = new InjectionToken<Logger>('LoggerToken');
```

Providing a Concrete Implementation:

```ts
@NgModule({
  providers: [
    { provide: LOGGER_TOKEN, useClass: ConsoleLogger },
  ],
})
export class AppModule {}
```

Injecting the Token:

```ts
export class MyComponent {
  constructor(@Inject(LOGGER_TOKEN) private logger: Logger) {
    this.logger.log('Logger initialized!');
  }
}
```


* * *

## **How Providers Work**

Providers are instructions for Angular's DI system to create and supply a dependency when requested. Understanding providers is critical for configuring DI, both in applications and in unit tests.

### **Types of Providers**

1.  **Class Provider:** Default. Uses a class to instantiate a service.

```ts
{ provide: DataService, useClass: DataService }
```

2.  **Value Provider:** Supplies a constant or object value.

```ts
{ provide: SOME_TOKEN, useValue: 'Some Value' }
```

3.  **Factory Provider:** Calls a factory function to produce the value.

```ts
{ provide: DataService, useFactory: () => new MockDataService() }
```

4.  **Existing Provider:** Redirects to another provider.

```ts
{ provide: SOME_TOKEN, useExisting: ANOTHER_TOKEN }
```

### **How Providers Work Outside Unit Tests**

*   In regular Angular applications, providers can be declared at different levels of the application:
    *   **Root-Level Providers:** Registered in `@Injectable({ providedIn: 'root' })` or `AppModule`. These are singleton services.
    *   **Component-Level Providers:** Declared in the `providers` array of a specific component. These create separate instances of the service for that component and its children.

Example:

```ts
@Component({
  selector: 'app-child',
  template: '<div></div>',
  providers: [{ provide: DataService, useClass: ChildDataService }],
})
export class ChildComponent {
  constructor(private dataService: DataService) {}
}
```

* * *

## **Dependency Injection (DI) in Unit Testing**

In unit tests, DI allows us to provide mock implementations of services and objects. This ensures that:
*   Tests are isolated and do not depend on real services or external dependencies (e.g., HTTP calls).
*   The test focuses on the component's behavior rather than the service's implementation.

DI plays a vital role in unit tests by allowing you to:
*   Provide mock implementations of services and objects to keep tests isolated.
*   Replace real dependencies (e.g., HTTP services) with stubs, spies, or mock versions.
Angular's **TestBed** is a utility that helps configure DI for unit testing. It allows you to define mock services, override providers, and inject dependencies as needed.

* * *

## **Inject vs. TestBed.inject**

Angular provides two ways to retrieve dependencies in tests:
| Feature | `inject()` | `TestBed.inject()` |
| --- | --- | --- |
| **Scope** | Limited to the test or function block. | Accessible globally in setup or teardown logic. |
| **Context** | Requires a callback function. | Can be called directly anywhere. |
| **API Simplicity** | Requires explicit dependency declaration in the callback. | Direct retrieval without extra boilerplate. |
| **Use Case** | Inline testing within `it()` blocks. | Complex setups, reusable logic. |

Example of `TestBed.inject`:

```ts
beforeEach(() => {
  const dataService = TestBed.inject(DataService);
  spyOn(dataService, 'getData').and.returnValue(['mockData']);
});
```
Example of `inject`:
```ts
it('should use the mock service', inject([DataService], (service: DataService) => {
  spyOn(service, 'getData').and.returnValue(['mockData']);
  expect(service.getData()).toEqual(['mockData']);
}));
```

* * *


## **Setting Up DI in Tests**

The `TestBed` utility in Angular is used to configure and initialize DI for tests. We can define mock services, override existing providers, and test isolated components.

### **Example: Setting Up DI for a Component Test**

Suppose you have a `UserService` that fetches user data, and a `ProfileComponent` that depends on it. Here's how you can set up DI in a test using `TestBed`:

##### **Original Implementation**

```ts
@Injectable()
export class UserService {
  getUser() {
    return { name: 'John Doe', age: 30 };
  }
}

@Component({
  selector: 'app-profile',
  template: '<div>{{ user?.name }}</div>',
})
export class ProfileComponent {
  user: { name: string; age: number } | undefined;

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.user = this.userService.getUser();
  }
}
```

#### **Testing the Component with DI**

In the test, we’ll use `TestBed` to configure DI and provide a mock implementation of the `UserService`.

```ts
import { TestBed } from '@angular/core/testing';
import { ProfileComponent } from './profile.component';
import { UserService } from './user.service';

describe('ProfileComponent', () => {
  let component: ProfileComponent;
  let mockUserService: jasmine.SpyObj<UserService>;

  beforeEach(() => {
    // Create a mock UserService with Jasmine
    mockUserService = jasmine.createSpyObj('UserService', ['getUser']);
    mockUserService.getUser.and.returnValue({ name: 'Mock User', age: 25 });

    TestBed.configureTestingModule({
      declarations: [ProfileComponent],
      providers: [
        { provide: UserService, useValue: mockUserService }, // Override the real UserService
      ],
    });

    // Create an instance of the component with DI
    component = TestBed.createComponent(ProfileComponent).componentInstance;
  });

  it('should fetch and display the user from the mock service', () => {
    // Trigger ngOnInit to fetch the user
    component.ngOnInit();

    // Assert the expected behavior
    expect(component.user).toEqual({ name: 'Mock User', age: 25 });
    expect(mockUserService.getUser).toHaveBeenCalled();
  });
});

```


* * *

## **Modern Usage with the `inject` Function**

The `inject` function simplifies dependency injection in tests by directly providing dependencies in test cases. It's a concise and powerful alternative to explicitly setting up providers in `TestBed` for quick, one-off use cases.

#### Example of `inject` in Tests:

```ts
import { inject } from '@angular/core/testing';
import { MyService } from './my-service';

describe('MyService', () => {
  it('should call getData correctly', inject([MyService], (service: MyService) => {
    spyOn(service, 'getData').and.returnValue(['mockData']);
    expect(service.getData()).toEqual(['mockData']);
  }));
});

```

This approach is especially useful for small, isolated tests or for injecting multiple dependencies in a single test.

* * *

## **Conclusion**

Dependency Injection is the cornerstone of Angular's modularity and testability. By mastering DI, including modern trends like the `inject` function, you'll be able to write effective, efficient, and maintainable unit tests for your Angular applications. This foundation is crucial before exploring advanced unit testing techniques or tackling complex scenarios.


### **Key Takeaways**

1.  **Mocking Dependencies**: By overriding the real `UserService` with a mock, we isolate the `ProfileComponent` to focus on its behavior.
2.  **Using `TestBed`**: The `TestBed.configureTestingModule` method sets up the testing environment, including providers, declarations, and imports.
3.  **Spy Integration**: Jasmine spies are used to mock methods and verify calls, ensuring expected interactions with the mock service.

