
- [Using Spies in Jasmine for Angular Unit Tests](#using-spies-in-jasmine-for-angular-unit-tests)
- [Key Concepts and Functions in Jasmine Spies](#key-concepts-and-functions-in-jasmine-spies)
  - [**Difference Between `spyOn`, `jasmine.createSpy`, `jasmine.createSpyOI` `spyOn`, `spyOnProperty`, and `spyOnAllFunctions`**](#difference-between-spyon-jasminecreatespy-jasminecreatespyoi-spyon-spyonproperty-and-spyonallfunctions)
      - [Example: Spying on a Getter](#example-spying-on-a-getter)
      - [Example: Spying on a Setter](#example-spying-on-a-setter)
      - [Example: Spying on All Methods of a Service](#example-spying-on-all-methods-of-a-service)
      - [Benefits of `spyOnAllFunctions`:](#benefits-of-spyonallfunctions)
      - [Caveats:](#caveats)
    - [**Best Practices for Reliable `jasmine.createSpyObj` Usage**](#best-practices-for-reliable-jasminecreatespyobj-usage)
    - [Comparison Summary](#comparison-summary)
- [Examples of Using Spies in Jasmine for Angular Unit Tests](#examples-of-using-spies-in-jasmine-for-angular-unit-tests)
  - [1. **Basic Spy on a Method**](#1-basic-spy-on-a-method)
    - [Example: Spy on a Method in a Component](#example-spy-on-a-method-in-a-component)
    - [Explanation:](#explanation)
  - [2. **Spying on a Service Method**](#2-spying-on-a-service-method)
    - [Example: Spying on a Service Method](#example-spying-on-a-service-method)
    - [Explanation:](#explanation-1)
  - [3. **Spying on HTTP Requests**](#3-spying-on-http-requests)
    - [Example: Spying on HttpClient's GET Method](#example-spying-on-httpclients-get-method)
    - [Explanation:](#explanation-2)
  - [4. **Spying on a Complex Service with Multiple Methods**](#4-spying-on-a-complex-service-with-multiple-methods)
    - [Example: Spying on Multiple Methods of a Service](#example-spying-on-multiple-methods-of-a-service)
    - [Explanation:](#explanation-3)
  - [5. **Spying on Methods with Arguments**](#5-spying-on-methods-with-arguments)
    - [Example: Spying with Arguments](#example-spying-with-arguments)
    - [Explanation:](#explanation-4)
  - [6. **Spying on an Injected Service Method**](#6-spying-on-an-injected-service-method)
  - [7. **Spying on Properties**](#7-spying-on-properties)
  - [8. **Spying with `jasmine.createSpy` on Properties**](#8-spying-with-jasminecreatespy-on-properties)
  - [9. **Spying on Functions in Modules**](#9-spying-on-functions-in-modules)
  - [10. **Spying on Asynchronous Methods**](#10-spying-on-asynchronous-methods)
  - [11. **Spying with CallThrough**](#11-spying-with-callthrough)
- [Conclusion](#conclusion)

---

# Using Spies in Jasmine for Angular Unit Tests

In Jasmine, **spies** are a powerful tool for testing. A **spy** allows you to track function calls and control behavior for the purpose of testing. It can replace a method, track how many times it's called, check with what arguments it was called, and even simulate its return value or behavior without needing the actual implementation.
Using spies in Jasmine tests is important in Angular unit testing because:
*   **Isolation:** Spies allow you to isolate the unit of code under test by replacing complex or external dependencies.
*   **Mocking Methods:** You can simulate different behaviors of services, APIs, and other methods.
*   **Verifying Interactions:** Spies can verify that the correct methods are called with the right arguments.

# Key Concepts and Functions in Jasmine Spies

*   **`jasmine.createSpy()`**: Creates a simple spy function.
*   **`jasmine.createSpyObj()`**: Creates a spy object with multiple spy functions.
*   **`spyOn()`**: Replaces a method on an existing object with a spy.
*   **`and.returnValue()`**: Specifies what the spy should return when it is called.
*   **`and.callFake()`**: Specifies a custom implementation for the spy.
*   **`and.callThrough()`**: Makes the spy call the original method.
*   **`toHaveBeenCalled()`**: Asserts that the spy was called.
*   **`toHaveBeenCalledWith()`**: Asserts that the spy was called with specific arguments.

## **Difference Between `spyOn`, `jasmine.createSpy`, `jasmine.createSpyOI` `spyOn`, `spyOnProperty`, and `spyOnAllFunctions`**

1.  **`spyOn`:**
    *   **Purpose**: Spies on **existing methods** of an **object**. It allows you to mock or track function calls of real methods that are part of an existing object.
    
    *   **When to Use**:
        *   When you have an existing object or class and want to mock or track a specific method’s behavior.
        *   Ideal for spying on methods that are part of a service, component, or class.
    *   **Key Features**:
        *   Can only spy on methods that already exist.
        *   Allows you to mock method behavior using `and.returnValue()` or `and.callFake()`.
```ts
spyOn(service, 'methodName').and.returnValue(mockValue);
```
    
2.  **`jasmine.createSpy`:**
    *   **Purpose**: Creates a **standalone spy function** that doesn't belong to any object. It can be used to mock a method or just track function calls.
    *   **When to Use**:
        *   When you need to create a mock function to use as a standalone mock (e.g., passing a spy function as an argument in a test).
        *   Useful when you want to mock a specific behavior but do not have an existing object or method to spy on.
    *   **Key Features**:
        *   Does not require an existing object or method.
        *   Can be used to mock a single function, not an entire object.

```ts
const mySpy = jasmine.createSpy('mySpy').and.returnValue(mockValue);
```

3.  **`jasmine.createSpyObj`**

    *   **Purpose**: Creates an **entire mock object** with a set of spy methods. This is ideal when you need to mock a service or an object with multiple methods.
    
    *   **When to Use**:
        *   When you want to mock an entire object and spy on its methods at once (e.g., mocking a service with multiple methods in a test).
        *   It's useful for testing scenarios where you expect multiple method calls on the object and need to mock all of them.
    *   **Key Features**:
        *   Automatically creates a mock object with the methods defined.
        *   Requires method names to be passed as an array.

```ts
const myMockObj = jasmine.createSpyObj('MyMock', ['methodA', 'methodB']);
```


4.  **`spyOnProperty`** 
    *   **Purpose**: Spies on **getter or setter properties** of an object.
    
    *   **When to Use**:
        *   When you need to mock or track the access to a **property** rather than a method (i.e., a getter or setter).
        *   Useful for objects with properties that are accessed frequently in tests (e.g., mock object attributes).
    *   **Key Features**:
        *   Works with **getter** and **setter** properties.
        *   You can spy on property access without directly spying on methods.

#### Example: Spying on a Getter
```ts
class MyClass {
  get myProperty() {
    return 'original value';
  }
}

describe('spyOnProperty example', () => {
  it('should mock the getter', () => {
    const instance = new MyClass();
    spyOnProperty(instance, 'myProperty', 'get').and.returnValue('mocked value');

    expect(instance.myProperty).toBe('mocked value');
  });
});

```
#### Example: Spying on a Setter
```ts
class MyClass {
  private _myProperty: string = 'default';

  set myProperty(value: string) {
    this._myProperty = value;
  }

  get myProperty() {
    return this._myProperty;
  }
}

describe('spyOnProperty example - setter', () => {
  it('should spy on the setter', () => {
    const instance = new MyClass();
    const spy = spyOnProperty(instance, 'myProperty', 'set').and.callFake((value: string) => {
      console.log(`Setter called with: ${value}`);
    });

    instance.myProperty = 'new value';
    expect(spy).toHaveBeenCalledWith('new value');
  });
});

```

5.  **`spyOnAllFunctions`** 
    *   **Purpose**: Spies on **all functions** of an **object** at once.
    
    *   **When to Use**:
        *   When you want to mock or track **all method calls** of an object without spying on each individual method.
        *   Useful when you need to track a wide range of method invocations on an object, especially for large objects with many methods.
    *   **Key Features**:
        *   Automatically spies on all functions of an object.
        *   Does not work for properties—only for methods.

#### Example: Spying on All Methods of a Service
```ts
class MyService {
  methodA() {
    return 'methodA';
  }

  methodB() {
    return 'methodB';
  }
}

describe('spyOnAllFunctions example', () => {
  it('should spy on all methods of a service', () => {
    const service = new MyService();
    const spyObject = spyOnAllFunctions(service);

    service.methodA();
    service.methodB();

    expect(spyObject.methodA).toHaveBeenCalled();
    expect(spyObject.methodB).toHaveBeenCalled();
  });
});

```
#### Benefits of `spyOnAllFunctions`:

*   Saves time by avoiding repetitive calls to `spyOn` for each individual method.
*   Provides a single point of control for all method spies within an object.

#### Caveats:

*   Be cautious when the object has methods that interact with critical dependencies, as spying on everything can obscure which methods are truly being called in specific test scenarios.



### **Best Practices for Reliable `jasmine.createSpyObj` Usage**

1.  **Mock Everything Explicitly**
    *   Be intentional with methods and properties you mock. Review the dependency and include everything your test will use.
2.  **Use TypeScript to Ensure Completeness**
    *   Use `keyof` to dynamically generate the list of methods to mock, reducing human error.
```ts
const methodsToMock = Object.keys(MyService.prototype) as Array<keyof MyService>;
const mockService = jasmine.createSpyObj('MyService', methodsToMock);
```


3.  **Validate Mock Behavior**
    *   Use assertions (`expect`) to ensure your mocks are behaving as intended.
```ts
expect(mockService.method1).toHaveBeenCalledWith(expectedArg);
expect(mockService.method1.calls.count()).toBe(1);
```

4.  **Organize Setup Logic**
    *   Centralize mock creation in `beforeEach` blocks for reuse but tailor it for specific tests when necessary.
5.  **Avoid Mocking Too Much**
    *   Consider using the actual implementation if mocking becomes overly complex. Use `spyOn` for individual methods instead of replacing the whole object.


### Comparison Summary

| **Feature** | `jasmine.createSpy` | `jasmine.createSpyObj` | `spyOn` | `spyOnProperty` | `spyOnAllFunctions` |
| --- | --- | --- | --- | --- | --- |
| **Object Required?** | No | Yes | Yes | Yes | Yes |
| **Purpose** | Mock a standalone function | Mock an entire object with methods | Mock an existing method | Mock getter/setter property | Mock all methods of an object |
| **Tracks Calls?** | Yes | Yes | Yes | Yes | Yes |
| **Overrides Behavior?** | Yes | Yes | Yes | Yes | Yes |
| **Use Case** | When no object exists | Mocking a service or object with multiple methods | Spying on an existing method of an object | Spying on getter/setter properties | Spying on all methods in an object |


# Examples of Using Spies in Jasmine for Angular Unit Tests

* * *

## 1. **Basic Spy on a Method**

The most basic use case for spies is replacing a function or method with a spy to track its calls or to simulate its return value.

### Example: Spy on a Method in a Component

```ts
import { Component } from '@angular/core';
import { TestBed } from '@angular/core/testing';

@Component({
  selector: 'app-test',
  template: '<button (click)="onClick()">Click me</button>'
})
export class TestComponent {
  message = 'Hello, World!';
  
  onClick() {
    this.message = 'Button Clicked!';
  }
}

describe('TestComponent', () => {
  let component: TestComponent;
  let spyOnClick: jasmine.Spy;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [TestComponent]
    });

    const fixture = TestBed.createComponent(TestComponent);
    component = fixture.componentInstance;

    // Creating a spy for onClick method
    spyOnClick = spyOn(component, 'onClick');
  });

  it('should call onClick when button is clicked', () => {
    // Triggering the button click
    component.onClick();

    // Assert that the spy was called
    expect(spyOnClick).toHaveBeenCalled();
  });
});

```

### Explanation:

*   The `spyOn(component, 'onClick')` spies on the `onClick` method of the `TestComponent`.
*   `expect(spyOnClick).toHaveBeenCalled()` asserts that the method was called when triggered.

## 2. **Spying on a Service Method**

In Angular tests, you often need to mock service methods using spies to isolate your component from the actual service logic.

### Example: Spying on a Service Method

```ts
import { Component } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { UserService } from './user.service'; // Assume this service is imported

@Component({
  selector: 'app-user',
  template: '<div>{{ user.name }}</div>'
})
export class UserComponent {
  user: any;

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.userService.getUser().subscribe(data => {
      this.user = data;
    });
  }
}

describe('UserComponent', () => {
  let component: UserComponent;
  let userServiceSpy: jasmine.SpyObj<UserService>;

  beforeEach(() => {
    // Create a spy for UserService
    userServiceSpy = jasmine.createSpyObj('UserService', ['getUser']);
    
    // Set up a mock return value
    userServiceSpy.getUser.and.returnValue(of({ name: 'John Doe' }));

    TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [{ provide: UserService, useValue: userServiceSpy }]
    });

    const fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
  });

  it('should call getUser on ngOnInit and populate user data', () => {
    component.ngOnInit();

    // Ensure getUser was called
    expect(userServiceSpy.getUser).toHaveBeenCalled();

    // Ensure the user data is updated
    expect(component.user.name).toBe('John Doe');
  });
});

```

### Explanation:

*   **Mocking Service:** `jasmine.createSpyObj('UserService', ['getUser'])` creates a spy for the `getUser` method of the `UserService`.
*   **Mock Return Value:** `userServiceSpy.getUser.and.returnValue(of({ name: 'John Doe' }))` simulates the return of an observable.
*   **Assertions:** The test checks that the `getUser` method was called and that the `user` object was populated correctly.

## 3. **Spying on HTTP Requests**

A common use case in Angular unit testing is mocking HTTP requests using spies. This allows you to test the behavior of components without making real HTTP calls.

### Example: Spying on HttpClient's GET Method

```ts
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { HttpClient } from '@angular/common/http';
import { UserService } from './user.service';

describe('UserService', () => {
  let userService: UserService;
  let httpClientSpy: jasmine.SpyObj<HttpClient>;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    httpClientSpy = jasmine.createSpyObj('HttpClient', ['get']);
    userService = new UserService(httpClientSpy);

    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });

    httpMock = TestBed.inject(HttpTestingController);
  });

  it('should return user data from API', () => {
    const mockUser = { id: 1, name: 'John Doe' };

    httpClientSpy.get.and.returnValue(of(mockUser));  // Mocking HTTP GET request

    userService.getUser().subscribe(user => {
      expect(user).toEqual(mockUser);
    });

    expect(httpClientSpy.get).toHaveBeenCalledWith('https://api.example.com/user');
  });
});

```

### Explanation:

*   **Spying on HTTP Calls:** `httpClientSpy = jasmine.createSpyObj('HttpClient', ['get'])` creates a spy for the `HttpClient` service.
*   **Mocking Response:** `httpClientSpy.get.and.returnValue(of(mockUser))` simulates an HTTP request returning an observable with the mock user data.
*   **Assertions:** The test checks that the `get` method was called with the correct URL and that the response was handled correctly.

## 4. **Spying on a Complex Service with Multiple Methods**

Sometimes you need to spy on multiple methods within a service. For instance, a service might have methods for `add`, `delete`, and `update` operations.

### Example: Spying on Multiple Methods of a Service

```ts
import { Component } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { TodoService } from './todo.service';

@Component({
  selector: 'app-todo',
  template: '<button (click)="addTodo()">Add Todo</button>'
})
export class TodoComponent {
  constructor(private todoService: TodoService) {}

  addTodo() {
    this.todoService.add('New Todo');
  }
}

describe('TodoComponent', () => {
  let component: TodoComponent;
  let todoServiceSpy: jasmine.SpyObj<TodoService>;

  beforeEach(() => {
    todoServiceSpy = jasmine.createSpyObj('TodoService', ['add', 'delete', 'update']);

    TestBed.configureTestingModule({
      declarations: [TodoComponent],
      providers: [{ provide: TodoService, useValue: todoServiceSpy }]
    });

    const fixture = TestBed.createComponent(TodoComponent);
    component = fixture.componentInstance;
  });

  it('should call add method of TodoService when addTodo is triggered', () => {
    component.addTodo();

    // Ensure add method was called with the correct argument
    expect(todoServiceSpy.add).toHaveBeenCalledWith('New Todo');
  });

  it('should check if the delete method is called', () => {
    component.deleteTodo();

    expect(todoServiceSpy.delete).toHaveBeenCalled();
  });

  it('should check if the update method is called', () => {
    component.updateTodo();

    expect(todoServiceSpy.update).toHaveBeenCalled();
  });
});

```

### Explanation:

*   **Spying on Multiple Methods:** `jasmine.createSpyObj('TodoService', ['add', 'delete', 'update'])` creates a spy object for the `TodoService` with multiple methods.
*   **Assertions:** Each test verifies that the corresponding method (`add`, `delete`, `update`) was called.

## 5. **Spying on Methods with Arguments**

Spies can also track and verify that a method is called with the correct arguments.

### Example: Spying with Arguments
```ts
import { TestBed } from '@angular/core/testing';
import { MyService } from './my.service';

describe('MyService', () => {
  let myServiceSpy: jasmine.SpyObj<MyService>;

  beforeEach(() => {
    myServiceSpy = jasmine.createSpyObj('MyService', ['saveData']);
    
    TestBed.configureTestingModule({
      providers: [{ provide: MyService, useValue: myServiceSpy }]
    });
  });

  it('should call saveData with the correct data', () => {
    const data = { name: 'Test Data' };
    myServiceSpy.saveData(data);

    expect(myServiceSpy.saveData).toHaveBeenCalledWith(data);
  });
});

```

### Explanation:

*   **Arguments Tracking:** The test verifies that the `saveData` method is called with the expected `data` object.

## 6. **Spying on an Injected Service Method**
```ts
import { TestBed } from '@angular/core/testing';
import { MyComponent } from './my-component';
import { MyService } from './my-service';

describe('MyComponent', () => {
  let component: MyComponent;
  let service: MyService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [MyComponent],
      providers: [MyService],
    });

    service = TestBed.inject(MyService);
    component = TestBed.createComponent(MyComponent).componentInstance;
  });

  it('should call MyService.getData()', () => {
    // Spy on the service method
    spyOn(service, 'getData').and.returnValue(['mockData']);

    // Act
    component.fetchData();

    // Assert
    expect(service.getData).toHaveBeenCalled();
    expect(component.data).toEqual(['mockData']);
  });
});

```

## 7. **Spying on Properties**

In Jasmine, you can spy on **properties** of objects using `spyOnProperty`. This is useful when you want to mock a getter or setter.

```ts
it('should spy on a property', () => {
  const obj = {
    get prop() {
      return 'original value';
    },
  };

  // Spy on the getter of the property
  spyOnProperty(obj, 'prop', 'get').and.returnValue('mocked value');

  expect(obj.prop).toBe('mocked value');
});

```

## 8. **Spying with `jasmine.createSpy` on Properties**

To spy on properties using `jasmine.createSpy`, you can use a getter or setter function in a class or service. However, unlike `spyOnProperty`, `createSpy` does not directly work on properties; it requires wrapping.

```ts
class MyClass {
  get myProp() {
    return 'real value';
  }
}

it('should use createSpy to mock a property getter', () => {
  const obj = new MyClass();

  // Create a spy for the getter
  const spyGetter = jasmine.createSpy('myPropGetter').and.returnValue('mocked value');

  // Override the getter with the spy
  Object.defineProperty(obj, 'myProp', { get: spyGetter });

  expect(obj.myProp).toBe('mocked value');
  expect(spyGetter).toHaveBeenCalled();
});

```

## 9. **Spying on Functions in Modules**
You can spy on global or module-level functions by importing the module and using `spyOn`.
```ts
import * as MathModule from './math-module';

it('should spy on a module function', () => {
  spyOn(MathModule, 'add').and.returnValue(42);

  expect(MathModule.add(1, 2)).toBe(42);
});

```

## 10. **Spying on Asynchronous Methods**
Use `spyOn` to mock methods that return promises or observables.

```ts
it('should mock an async method returning a promise', async () => {
  const asyncSpy = jasmine.createSpy('asyncMethod').and.resolveTo('mocked value');

  const result = await asyncSpy();

  expect(result).toBe('mocked value');
});

```

## 11. **Spying with CallThrough**
If you want the original implementation to still execute while monitoring it, use `.and.callThrough()`.

```ts
it('should call the original implementation', () => {
  const obj = {
    method: () => 'original value',
  };

  spyOn(obj, 'method').and.callThrough();

  expect(obj.method()).toBe('original value');
  expect(obj.method).toHaveBeenCalled();
});

```


* * *

# Conclusion

Spies are a vital part of testing in Angular, especially when you need to mock methods, track function calls, or isolate dependencies. Using spies effectively can help you:
*   Mock services or methods to simulate specific behaviors.
*   Verify that the correct methods are called with the expected arguments.
*   Improve test isolation, making unit tests faster and more reliable.
Spies are particularly useful when testing Angular components that rely on external services, APIs, or complex logic that you'd prefer to isolate during testing.
