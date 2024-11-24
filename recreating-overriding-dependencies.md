Table of content

# Introduction:

In Angular unit tests, **dependencies** refer to services, components, or other objects that a component or service relies on. These dependencies are typically injected into the component or service using Angular's **Dependency Injection (DI)** system. During testing, you might want to **recreate** or **override** these dependencies to ensure the test runs in isolation and to control the behavior of external services or components.
**Recreating** a dependency means to replace it with a fresh instance, while **overriding** means replacing it with a mock or a different implementation for testing purposes.
These strategies are especially useful for:
*   **Mocking Services**: Replace real services with mock implementations to avoid calling real APIs, databases, or external services.
*   **Controlling Behavior**: Mocking specific methods or properties of services to test different scenarios.
*   **Test Isolation**: Ensuring tests do not depend on each other and run in isolation by controlling the behavior of shared dependencies.

### Key Approaches

1.  **Recreating Dependencies**
    *   This involves replacing a service or other dependency with a fresh instance for each test.
    *   You can recreate dependencies by modifying the `TestBed` configuration.
2.  **Overriding Dependencies**
    *   This means replacing the default dependency with a mock or a custom implementation for the purpose of the test.
Let’s go through examples to demonstrate both approaches.

* * *

# 1. **Recreating Dependencies**

In Angular, when you configure a provider using `TestBed`, you can ensure that every test gets a fresh instance of the dependency by setting it up in `beforeEach` or `beforeAll`.

### Example: Recreating a Service Dependency for Each Test

Imagine you have a `UserService` that fetches user data, and you want to make sure each test gets a fresh instance of the service.

```ts
import { TestBed } from '@angular/core/testing';
import { UserComponent } from './user.component';
import { UserService } from './user.service';

describe('UserComponent', () => {

  let component: UserComponent;
  let userService: UserService;

  // Setup TestBed for each test
  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [UserService] // Each test will get a fresh instance
    });

    // Create a fresh instance of UserService
    userService = TestBed.inject(UserService);
    component = TestBed.createComponent(UserComponent).componentInstance;
  });

  it('should create the component and service instance', () => {
    expect(component).toBeTruthy();
    expect(userService).toBeTruthy(); // Ensures fresh instance
  });

});

```

### **Explanation**:

*   The `TestBed.configureTestingModule()` ensures that each test gets a new instance of `UserService`.
*   In `beforeEach`, `TestBed.inject(UserService)` is called to get a fresh instance of the service each time.

# 2. **Overriding Dependencies (Mocking)**

In many unit tests, you may want to mock or override services to avoid interacting with external resources (e.g., APIs or databases). You can do this by using **mock services** or **spies**.

### Example: Overriding a Service with a Mock

In the following example, instead of using the real `UserService`, we provide a mock implementation of the service to isolate the test case.

```ts
import { TestBed } from '@angular/core/testing';
import { UserComponent } from './user.component';
import { UserService } from './user.service';
import { of } from 'rxjs';

describe('UserComponent with Mocked Service', () => {

  let component: UserComponent;
  let mockUserService: jasmine.SpyObj<UserService>;

  beforeEach(() => {
    // Create a mock service
    mockUserService = jasmine.createSpyObj('UserService', ['getUser']);

    // Setup the mock service's behavior
    mockUserService.getUser.and.returnValue(of({ id: 1, name: 'John Doe' }));

    // Configure the TestBed to use the mock service
    TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [{ provide: UserService, useValue: mockUserService }] // Overriding the provider
    });

    // Create the component instance
    component = TestBed.createComponent(UserComponent).componentInstance;
  });

  it('should use the mocked user service', () => {
    // Trigger ngOnInit or the method that uses the service
    component.ngOnInit();

    // Assert that the mock service was called
    expect(mockUserService.getUser).toHaveBeenCalled();

    // Check if the component has the mocked user data
    expect(component.user).toEqual({ id: 1, name: 'John Doe' });
  });

});

```

### **Explanation:**

*   **Mocking the Service:** A mock of `UserService` is created using `jasmine.createSpyObj()` and provided using `useValue` in `TestBed.configureTestingModule()`.
*   **Controlling Behavior:** The mock `getUser()` method is set to return an observable with a mock user object. This allows us to test the component without calling the actual service or API.


# 3. **Overriding Services with `TestBed.overrideProvider()`**

In cases where you need to change the provider for a service after initial setup (for example, in the middle of a test suite), you can use `TestBed.overrideProvider()` to modify the provider.

### Example: Dynamically Overriding a Provider

```ts
import { TestBed } from '@angular/core/testing';
import { UserComponent } from './user.component';
import { UserService } from './user.service';
import { of } from 'rxjs';

describe('UserComponent with Dynamic Override', () => {

  let component: UserComponent;
  let mockUserService: jasmine.SpyObj<UserService>;

  beforeEach(() => {
    // Create the mock service
    mockUserService = jasmine.createSpyObj('UserService', ['getUser']);
    
    // Initial setup with the original service
    TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [UserService] // Use the original UserService
    });

    component = TestBed.createComponent(UserComponent).componentInstance;
  });

  it('should use the real service initially', () => {
    // Inject the real service initially
    const userService = TestBed.inject(UserService);
    expect(userService).toBeTruthy();
  });

  it('should override the service with a mock', () => {
    // Override the provider to use the mock service
    TestBed.overrideProvider(UserService, { useValue: mockUserService });
    mockUserService.getUser.and.returnValue(of({ id: 2, name: 'Jane Doe' }));

    // Recreate the component to use the overridden service
    component = TestBed.createComponent(UserComponent).componentInstance;

    component.ngOnInit();

    // Assert that the mock service was used
    expect(mockUserService.getUser).toHaveBeenCalled();
    expect(component.user).toEqual({ id: 2, name: 'Jane Doe' });
  });

});

```

### **Explanation:**

*   **Dynamic Override:** In the second test case, we override the `UserService` provider with a mock using `TestBed.overrideProvider()`.
*   **Test Isolation:** This allows us to test how the component behaves when the service is mocked, while the first test uses the real service.

# 4. **Recreating and Overriding Multiple Dependencies**

In some tests, you may want to recreate or override several services at once.

### Example: Overriding Multiple Dependencies
```ts
import { TestBed } from '@angular/core/testing';
import { UserComponent } from './user.component';
import { UserService } from './user.service';
import { AuthService } from './auth.service';
import { of } from 'rxjs';

describe('UserComponent with Multiple Mocks', () => {

  let component: UserComponent;
  let mockUserService: jasmine.SpyObj<UserService>;
  let mockAuthService: jasmine.SpyObj<AuthService>;

  beforeEach(() => {
    // Create mock services
    mockUserService = jasmine.createSpyObj('UserService', ['getUser']);
    mockAuthService = jasmine.createSpyObj('AuthService', ['isAuthenticated']);

    // Setup default behaviors
    mockUserService.getUser.and.returnValue(of({ id: 1, name: 'John Doe' }));
    mockAuthService.isAuthenticated.and.returnValue(of(true));

    // Configure the TestBed to use the mock services
    TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [
        { provide: UserService, useValue: mockUserService },
        { provide: AuthService, useValue: mockAuthService }
      ]
    });

    component = TestBed.createComponent(UserComponent).componentInstance;
  });

  it('should use both mocked services', () => {
    // Trigger component logic
    component.ngOnInit();

    // Assert that both mocks are called
    expect(mockUserService.getUser).toHaveBeenCalled();
    expect(mockAuthService.isAuthenticated).toHaveBeenCalled();

    // Assert that the component has the mocked user
    expect(component.user).toEqual({ id: 1, name: 'John Doe' });
  });

});

```

### **Explanation:**

*   **Multiple Mocks:** Both `UserService` and `AuthService` are mocked using `jasmine.createSpyObj()`. The mocks are injected via `TestBed` for the `UserComponent`.
*   **Isolated Tests:** This ensures that the component can be tested without hitting any real APIs or services.

# Summary of Key Concepts:

1.  **Recreating Dependencies:**
    *   Use `TestBed.configureTestingModule()` to configure fresh instances for each test.
    *   Ensure the components and services are isolated from each other by re-initializing dependencies.
2.  **Overriding Dependencies:**
    *   Use `TestBed.overrideProvider()` to replace services with mock implementations.
    *   Mocking allows you to control service behavior (e.g., for API calls, business logic, etc.) during unit tests.
3.  **Dynamic Configuration:**
    *   You can dynamically modify dependencies inside specific tests to isolate different behaviors.
    *   This is useful when testing components with complex dependencies, such as external services or configurations.
By using these strategies, you can isolate components and services during tests, ensuring that your unit tests are fast, reliable, and focused only on the logic of the component you're testing.
