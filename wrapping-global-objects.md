- [Wrapping Global Objects in Angular Unit Tests with Jasmine](#wrapping-global-objects-in-angular-unit-tests-with-jasmine)
- [Why Use Global Object Wrapping?](#why-use-global-object-wrapping)
- [Example 1: Wrapping the `window` Object](#example-1-wrapping-the-window-object)
  - [Step 1: Create a Window Wrapper Service](#step-1-create-a-window-wrapper-service)
  - [Step 2: Inject and Use the Service in a Component](#step-2-inject-and-use-the-service-in-a-component)
  - [Step 3: Test the Component with Jasmine and Spies](#step-3-test-the-component-with-jasmine-and-spies)
    - [Explanation:](#explanation)
- [Example 2: Wrapping `localStorage`](#example-2-wrapping-localstorage)
  - [Step 1: Create a LocalStorage Wrapper Service](#step-1-create-a-localstorage-wrapper-service)
  - [Step 2: Use the Service in a Component](#step-2-use-the-service-in-a-component)
  - [Step 3: Write Unit Tests with Jasmine](#step-3-write-unit-tests-with-jasmine)
    - [Explanation:](#explanation-1)

---

# Wrapping Global Objects in Angular Unit Tests with Jasmine

In Angular unit tests, **wrapping global objects** like `window`, `document`, `localStorage`, or other global browser APIs can be useful when you need to isolate tests, mock behaviors, or replace specific parts of the browser environment. These global objects can be difficult to manipulate directly within unit tests because they aren't easily injectable or mockable in the standard Angular testing setup.
To solve this problem, **wrapping global objects** allows you to create an abstraction layer around them, making it easier to mock their behavior in tests.
One common approach is to wrap the `window` object in a service or a provider so that it can be injected and manipulated in unit tests.

# Why Use Global Object Wrapping?

1.  **Isolation:** Wrapping allows you to isolate the component under test from external global dependencies like `window`, which could affect the test behavior.
2.  **Testability:** Global objects like `window` are not easily injectable, so by wrapping them, you make them easier to mock in tests.
3.  **Control:** Wrapping helps you control and manipulate global objects like `localStorage` or `window` during testing to simulate different behaviors (e.g., screen sizes, geolocation, etc.).
4.  **Cross-Browser Testing:** Wrapping provides a layer of abstraction, which is especially useful if you need to handle cross-browser differences in unit tests.

# Example 1: Wrapping the `window` Object

One common approach is to wrap the `window` object in an injectable service, allowing you to mock or override its behavior in unit tests.

## Step 1: Create a Window Wrapper Service

First, create an interface and a service to wrap the global `window` object.
```ts
// window-wrapper.service.ts
import { Injectable } from '@angular/core';

export interface WindowWrapper {
  getWindow(): Window;
  setWidth(width: number): void;
  setHeight(height: number): void;
}

@Injectable({
  providedIn: 'root'
})
export class WindowWrapperService implements WindowWrapper {
  getWindow(): Window {
    return window;
  }

  setWidth(width: number): void {
    this.getWindow().innerWidth = width;
  }

  setHeight(height: number): void {
    this.getWindow().innerHeight = height;
  }
}

```

*   **`WindowWrapper` interface:** This defines the contract for interacting with the `window` object, including methods like `getWindow()`, `setWidth()`, and `setHeight()`.
*   **`WindowWrapperService` class:** This class wraps the actual `window` object and provides methods to manipulate properties like `innerWidth` and `innerHeight`.

## Step 2: Inject and Use the Service in a Component

Now, use the `WindowWrapperService` in a component.
```ts
// component.ts
import { Component, OnInit } from '@angular/core';
import { WindowWrapperService } from './window-wrapper.service';

@Component({
  selector: 'app-window-test',
  template: '<p>Window width: {{ windowWidth }}</p>',
})
export class WindowTestComponent implements OnInit {
  windowWidth: number;

  constructor(private windowWrapper: WindowWrapperService) {}

  ngOnInit() {
    this.windowWidth = this.windowWrapper.getWindow().innerWidth;
  }

  resizeWindow(newWidth: number) {
    this.windowWrapper.setWidth(newWidth);
    this.windowWidth = this.windowWrapper.getWindow().innerWidth;
  }
}

```

In this example, the `WindowTestComponent` uses the `WindowWrapperService` to interact with the `window` object, particularly the `innerWidth` property.

## Step 3: Test the Component with Jasmine and Spies

Now, let's write a Jasmine unit test where we mock the `window` object behavior using the `WindowWrapperService`.
```ts
import { TestBed } from '@angular/core/testing';
import { WindowWrapperService } from './window-wrapper.service';
import { WindowTestComponent } from './window-test.component';
import { ComponentFixture } from '@angular/core/testing';
import { of } from 'rxjs';

describe('WindowTestComponent', () => {
  let component: WindowTestComponent;
  let fixture: ComponentFixture<WindowTestComponent>;
  let windowWrapperSpy: jasmine.SpyObj<WindowWrapperService>;

  beforeEach(() => {
    windowWrapperSpy = jasmine.createSpyObj('WindowWrapperService', ['getWindow', 'setWidth', 'setHeight']);

    TestBed.configureTestingModule({
      declarations: [WindowTestComponent],
      providers: [
        { provide: WindowWrapperService, useValue: windowWrapperSpy }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent(WindowTestComponent);
    component = fixture.componentInstance;
  });

  it('should display window width', () => {
    // Mock the window's innerWidth value
    windowWrapperSpy.getWindow.and.returnValue({ innerWidth: 1024 } as Window);

    fixture.detectChanges();

    expect(component.windowWidth).toBe(1024);
  });

  it('should resize the window width when resizeWindow is called', () => {
    // Mock the window's innerWidth value
    windowWrapperSpy.getWindow.and.returnValue({ innerWidth: 1024 } as Window);
    component.resizeWindow(800);

    // Ensure the window width is updated
    expect(component.windowWidth).toBe(800);
    expect(windowWrapperSpy.setWidth).toHaveBeenCalledWith(800);
  });
});

```

### Explanation:

*   **Spying on `WindowWrapperService`:** We create a spy object for the `WindowWrapperService` and mock the behavior of the `getWindow` and `setWidth` methods.
*   **Mocking the `innerWidth`:** In the first test case, we mock the `getWindow` method to return a `window` object with a specific `innerWidth` value.
*   **Simulating `resizeWindow`:** In the second test, we call `resizeWindow`, which uses the `WindowWrapperService` to update the window's width and verify that the `setWidth` method was called correctly.

# Example 2: Wrapping `localStorage`

Another common global object you might want to wrap is `localStorage`. Direct access to `localStorage` can cause issues in tests if you need to manipulate or mock its behavior.

## Step 1: Create a LocalStorage Wrapper Service

```ts
// local-storage-wrapper.service.ts
import { Injectable } from '@angular/core';

export interface LocalStorageWrapper {
  setItem(key: string, value: string): void;
  getItem(key: string): string | null;
  removeItem(key: string): void;
}

@Injectable({
  providedIn: 'root'
})
export class LocalStorageWrapperService implements LocalStorageWrapper {
  setItem(key: string, value: string): void {
    localStorage.setItem(key, value);
  }

  getItem(key: string): string | null {
    return localStorage.getItem(key);
  }

  removeItem(key: string): void {
    localStorage.removeItem(key);
  }
}

```
## Step 2: Use the Service in a Component

```ts
// component.ts
import { Component, OnInit } from '@angular/core';
import { LocalStorageWrapperService } from './local-storage-wrapper.service';

@Component({
  selector: 'app-local-storage-test',
  template: '<p>{{ localStorageData }}</p>',
})
export class LocalStorageTestComponent implements OnInit {
  localStorageData: string | null = null;

  constructor(private localStorageService: LocalStorageWrapperService) {}

  ngOnInit() {
    this.localStorageData = this.localStorageService.getItem('myKey');
  }

  saveData(data: string) {
    this.localStorageService.setItem('myKey', data);
    this.localStorageData = data;
  }
}

```

## Step 3: Write Unit Tests with Jasmine

```ts
import { TestBed } from '@angular/core/testing';
import { LocalStorageWrapperService } from './local-storage-wrapper.service';
import { LocalStorageTestComponent } from './local-storage-test.component';
import { ComponentFixture } from '@angular/core/testing';

describe('LocalStorageTestComponent', () => {
  let component: LocalStorageTestComponent;
  let fixture: ComponentFixture<LocalStorageTestComponent>;
  let localStorageSpy: jasmine.SpyObj<LocalStorageWrapperService>;

  beforeEach(() => {
    localStorageSpy = jasmine.createSpyObj('LocalStorageWrapperService', ['setItem', 'getItem', 'removeItem']);

    TestBed.configureTestingModule({
      declarations: [LocalStorageTestComponent],
      providers: [
        { provide: LocalStorageWrapperService, useValue: localStorageSpy }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent(LocalStorageTestComponent);
    component = fixture.componentInstance;
  });

  it('should read from localStorage on init', () => {
    localStorageSpy.getItem.and.returnValue('Test Data');

    fixture.detectChanges();

    expect(component.localStorageData).toBe('Test Data');
  });

  it('should save data to localStorage', () => {
    component.saveData('New Data');

    expect(localStorageSpy.setItem).toHaveBeenCalledWith('myKey', 'New Data');
    expect(component.localStorageData).toBe('New Data');
  });
});

```

### Explanation:

*   **Spying on `LocalStorageWrapperService`:** The `localStorageSpy` object mocks the methods for interacting with `localStorage` (`setItem`, `getItem`, `removeItem`).
*   **Mocking Behavior:** In the first test, we mock `getItem` to return a specific value when called.
*   **Saving Data:** In the second test, we check that the `localStorage.setItem` method is called with the correct arguments
