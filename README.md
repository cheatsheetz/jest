# Jest Cheat Sheet

Jest is a JavaScript testing framework developed by Facebook, designed for testing React applications but works with any JavaScript project. It provides test runner, assertion library, mocking capabilities, and code coverage reports out of the box.

---

## Table of Contents
- [Installation & Setup](#installation--setup)
- [Basic Test Structure](#basic-test-structure)
- [Matchers & Assertions](#matchers--assertions)
- [Async Testing](#async-testing)
- [Mocking](#mocking)
- [Configuration](#configuration)
- [Code Coverage](#code-coverage)
- [CI/CD Integration](#cicd-integration)
- [Best Practices](#best-practices)

---

## Installation & Setup

### Installation
```bash
# Install Jest
npm install --save-dev jest

# For TypeScript support
npm install --save-dev @types/jest ts-jest

# For React testing
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

### Package.json Configuration
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

## Basic Test Structure

| Function | Description | Example |
|----------|-------------|---------|
| `describe()` | Group related tests | `describe('Calculator', () => {})` |
| `test()` / `it()` | Define individual test | `test('adds 1 + 2 to equal 3', () => {})` |
| `beforeEach()` | Run before each test | `beforeEach(() => { setup() })` |
| `afterEach()` | Run after each test | `afterEach(() => { cleanup() })` |
| `beforeAll()` | Run once before all tests | `beforeAll(() => { globalSetup() })` |
| `afterAll()` | Run once after all tests | `afterAll(() => { globalCleanup() })` |

### Basic Test Example
```javascript
// math.js
function add(a, b) {
  return a + b;
}
module.exports = add;

// math.test.js
const add = require('./math');

describe('Math functions', () => {
  test('adds 1 + 2 to equal 3', () => {
    expect(add(1, 2)).toBe(3);
  });
});
```

## Matchers & Assertions

### Common Matchers
| Matcher | Description | Example |
|---------|-------------|---------|
| `.toBe()` | Exact equality (===) | `expect(2 + 2).toBe(4)` |
| `.toEqual()` | Deep equality | `expect({name: 'John'}).toEqual({name: 'John'})` |
| `.toBeNull()` | Matches null | `expect(null).toBeNull()` |
| `.toBeUndefined()` | Matches undefined | `expect(undefined).toBeUndefined()` |
| `.toBeTruthy()` | Matches truthy values | `expect('hello').toBeTruthy()` |
| `.toBeFalsy()` | Matches falsy values | `expect('').toBeFalsy()` |

### String Matchers
```javascript
test('string matchers', () => {
  expect('Hello World').toMatch(/World/);
  expect('Hello World').toContain('World');
  expect('hello@example.com').toMatch(/\S+@\S+\.\S+/);
});
```

### Array & Object Matchers
```javascript
test('array and object matchers', () => {
  expect(['Alice', 'Bob', 'Eve']).toContain('Alice');
  expect([1, 2, 3]).toHaveLength(3);
  expect({name: 'John', age: 30}).toHaveProperty('name');
  expect({name: 'John', age: 30}).toHaveProperty('name', 'John');
});
```

### Number Matchers
```javascript
test('number matchers', () => {
  expect(2 + 2).toBeGreaterThan(3);
  expect(3.14).toBeCloseTo(3.1, 1);
  expect(Math.PI).toBeCloseTo(3.14159, 4);
});
```

## Async Testing

### Testing Promises
```javascript
// Using async/await
test('async test with async/await', async () => {
  const data = await fetchUser();
  expect(data.name).toBe('John');
});

// Using .resolves
test('async test with resolves', () => {
  return expect(fetchUser()).resolves.toHaveProperty('name', 'John');
});

// Testing rejections
test('async test rejection', async () => {
  await expect(fetchInvalidUser()).rejects.toThrow('User not found');
});
```

### Testing Callbacks
```javascript
test('callback test', (done) => {
  function callback(data) {
    try {
      expect(data).toBe('data');
      done();
    } catch (error) {
      done(error);
    }
  }
  fetchDataCallback(callback);
});
```

### Testing Timers
```javascript
// Enable fake timers
beforeEach(() => {
  jest.useFakeTimers();
});

afterEach(() => {
  jest.useRealTimers();
});

test('calls callback after 1 second', () => {
  const callback = jest.fn();
  setTimeout(callback, 1000);
  
  expect(callback).not.toBeCalled();
  jest.advanceTimersByTime(1000);
  expect(callback).toBeCalled();
});
```

## Mocking

### Function Mocking
```javascript
// Mock a function
const mockCallback = jest.fn();
mockCallback.mockReturnValue(42);
mockCallback.mockReturnValueOnce(10);

// Check mock calls
expect(mockCallback).toHaveBeenCalled();
expect(mockCallback).toHaveBeenCalledTimes(2);
expect(mockCallback).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockCallback).toHaveBeenLastCalledWith('lastArg');
```

### Module Mocking
```javascript
// Mock entire module
jest.mock('./api');
const api = require('./api');

// Mock specific function
jest.mock('./utils', () => ({
  calculateTax: jest.fn(() => 10),
  formatCurrency: jest.fn(x => `$${x}`)
}));

// Partial mocking
jest.mock('./config', () => ({
  ...jest.requireActual('./config'),
  API_URL: 'http://localhost'
}));
```

### Class Mocking
```javascript
// Mock ES6 class
jest.mock('./Database');
const Database = require('./Database');

beforeEach(() => {
  Database.mockClear();
});

test('should create database instance', () => {
  const db = new Database();
  expect(Database).toHaveBeenCalledTimes(1);
});
```

### Mock Implementation
```javascript
const mockFn = jest.fn();
mockFn.mockImplementation((x) => x * 2);

// Or use mockImplementationOnce
mockFn.mockImplementationOnce((x) => x * 3);
```

## Configuration

### jest.config.js
```javascript
module.exports = {
  // Test environment
  testEnvironment: 'node', // or 'jsdom' for browser-like environment
  
  // Test file patterns
  testMatch: ['**/__tests__/**/*.js', '**/*.test.js'],
  
  // Setup files
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  
  // Coverage configuration
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/index.js',
    '!**/node_modules/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  
  // Module name mapping
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  
  // Transform files
  transform: {
    '^.+\\.tsx?$': 'ts-jest'
  }
};
```

## Code Coverage

### Running Coverage
```bash
# Generate coverage report
npm test -- --coverage

# Coverage with threshold
npm test -- --coverage --coverageThreshold='{"global":{"lines":90}}'

# Watch mode with coverage
npm test -- --watch --coverage
```

### Coverage Configuration
```javascript
// In jest.config.js
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'html', 'lcov'],
  coverageThreshold: {
    global: {
      branches: 85,
      functions: 85,
      lines: 85,
      statements: 85
    },
    './src/components/': {
      branches: 90,
      statements: 90
    }
  }
};
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-node@v2
      with:
        node-version: '16'
    - run: npm ci
    - run: npm test -- --coverage --watchAll=false
    - uses: codecov/codecov-action@v2
```

### Jenkins Pipeline
```groovy
pipeline {
  agent any
  stages {
    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }
    stage('Test') {
      steps {
        sh 'npm test -- --coverage --watchAll=false --reporters=jest-junit'
      }
      post {
        always {
          junit 'junit.xml'
          publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'coverage/lcov-report',
            reportFiles: 'index.html',
            reportName: 'Coverage Report'
          ])
        }
      }
    }
  }
}
```

## Best Practices

### Test Structure
- **Arrange, Act, Assert**: Organize tests with clear setup, execution, and verification
- **One assertion per test**: Keep tests focused and easy to debug
- **Descriptive test names**: Use clear, descriptive names that explain what is being tested

```javascript
// Good
test('should return user name when valid ID is provided', () => {
  // Arrange
  const userId = 123;
  const expectedUser = { id: 123, name: 'John Doe' };
  
  // Act
  const result = getUserById(userId);
  
  // Assert
  expect(result.name).toBe(expectedUser.name);
});
```

### Mocking Guidelines
- **Mock external dependencies**: Always mock API calls, database connections, file system operations
- **Don't over-mock**: Only mock what you need to isolate the unit under test
- **Reset mocks**: Clear mocks between tests to avoid interference

### Performance Tips
- **Use beforeEach/afterEach wisely**: Avoid expensive operations in these hooks
- **Selective test running**: Use `--testNamePattern` or `--testPathPattern` for focused testing
- **Parallel testing**: Jest runs tests in parallel by default, avoid shared state

### Common Patterns
```javascript
// Test factory pattern
function createUser(overrides = {}) {
  return {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    ...overrides
  };
}

// Custom matcher
expect.extend({
  toBeValidEmail(received) {
    const pass = /\S+@\S+\.\S+/.test(received);
    return {
      message: () => `expected ${received} ${pass ? 'not ' : ''}to be a valid email`,
      pass,
    };
  },
});
```

---

## Resources
- [Jest Official Documentation](https://jestjs.io/docs/getting-started)
- [Testing Library Jest DOM](https://github.com/testing-library/jest-dom)
- [Jest Community Resources](https://github.com/facebook/jest/blob/main/website/community.md)
- [Testing JavaScript Course](https://testingjavascript.com/)

---
*Originally compiled from various sources. Contributions welcome!*