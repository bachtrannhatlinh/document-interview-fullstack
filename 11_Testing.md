# 11. Testing
- **Unit test**: Kiểm thử từng function/module nhỏ, đảm bảo logic hoạt động đúng độc lập.
  - Ví dụ (Jest):
    ```js
    // math.js
    function add(a, b) { return a + b; }
    module.exports = { add };
    // math.test.js
    const { add } = require('./math');
    test('add 2 + 3 = 5', () => {
      expect(add(2, 3)).toBe(5);
    });
    ```

- **Integration test**: Kiểm thử nhiều module/service kết hợp, thường test API endpoint.
  - Ví dụ (Supertest + Jest):
    ```js
    const request = require('supertest');
    const app = require('./app');
    test('GET /users trả về 200', async () => {
      const res = await request(app).get('/users');
      expect(res.statusCode).toBe(200);
    });
    ```

- **Mock**: Giả lập function/module/phụ thuộc bên ngoài (DB, API, ...), giúp test độc lập, không phụ thuộc môi trường thật.
  - Ví dụ (Jest):
    ```js
    jest.mock('./db');
    const db = require('./db');
    db.getUser.mockReturnValue({ id: 1, name: 'A' });
    ```

- **Coverage**: Đo lường tỷ lệ code được test (statement, branch, function, line).
  - Chạy với Jest: `jest --coverage`

## Testing Pyramid & E2E Testing
- **Testing Pyramid**: Unit tests (nhiều nhất) → Integration tests → E2E tests (ít nhất)
  - **Unit**: Nhanh, rẻ, dễ debug
  - **Integration**: Test tương tác giữa components
  - **E2E**: Test toàn bộ user journey, chậm nhưng tin cậy cao

- **End-to-End Testing:**
  ```js
  // Cypress example
  describe('User Login', () => {
    it('should login successfully', () => {
      cy.visit('/login');
      cy.get('[data-testid="email"]').type('user@example.com');
      cy.get('[data-testid="password"]').type('password123');
      cy.get('[data-testid="login-btn"]').click();
      cy.url().should('include', '/dashboard');
    });
  });

  // Playwright example
  test('user can login', async ({ page }) => {
    await page.goto('/login');
    await page.fill('#email', 'user@example.com');
    await page.fill('#password', 'password123');
    await page.click('#login-btn');
    await expect(page).toHaveURL(/.*dashboard/);
  });
  ```

## React/Frontend Testing
- **React Testing Library (RTL):**
  ```js
  import { render, screen, fireEvent } from '@testing-library/react';
  import Counter from './Counter';

  test('increments counter when clicked', () => {
    render(<Counter />);
    const button = screen.getByRole('button', { name: /increment/i });
    const count = screen.getByTestId('count');
    
    expect(count).toHaveTextContent('0');
    fireEvent.click(button);
    expect(count).toHaveTextContent('1');
  });
  ```

- **Component Testing Best Practices:**
  ```js
  // Test behavior, not implementation
  test('shows error message when form is invalid', async () => {
    render(<LoginForm />);
    
    fireEvent.click(screen.getByRole('button', { name: /login/i }));
    
    await waitFor(() => {
      expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    });
  });

  // Mock external dependencies
  jest.mock('./api');
  const mockApi = require('./api');
  
  test('displays user data after fetch', async () => {
    mockApi.fetchUser.mockResolvedValue({ id: 1, name: 'John' });
    
    render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
    });
  });
  ```

## Async Testing Patterns
- **Testing Promises:**
  ```js
  // Using async/await
  test('fetches user data', async () => {
    const userData = await fetchUser('123');
    expect(userData.name).toBe('John');
  });

  // Using resolves/rejects
  test('api call succeeds', () => {
    return expect(fetchUser('123')).resolves.toHaveProperty('name');
  });

  test('api call fails', () => {
    return expect(fetchUser('invalid')).rejects.toThrow('User not found');
  });
  ```

- **Testing Timers:**
  ```js
  jest.useFakeTimers();
  
  test('callback is called after timeout', () => {
    const callback = jest.fn();
    setTimeout(callback, 1000);
    
    jest.advanceTimersByTime(1000);
    expect(callback).toHaveBeenCalled();
  });
  ```

## Database Testing
- **Test Database Setup:**
  ```js
  // Setup test database
  beforeEach(async () => {
    await db.sync({ force: true });
    await seedTestData();
  });

  afterEach(async () => {
    await db.drop();
  });

  test('creates user in database', async () => {
    const user = await User.create({ 
      email: 'test@example.com',
      password: 'hashedpassword'
    });
    
    expect(user.id).toBeDefined();
    expect(user.email).toBe('test@example.com');
  });
  ```

- **Transaction Testing:**
  ```js
  test('rollback on error', async () => {
    const transaction = await db.transaction();
    
    try {
      await User.create({ email: 'test@test.com' }, { transaction });
      await User.create({ email: 'invalid-email' }, { transaction });
      await transaction.commit();
    } catch (error) {
      await transaction.rollback();
      
      const userCount = await User.count();
      expect(userCount).toBe(0);
    }
  });
  ```

## TDD (Test-Driven Development)
- **Red-Green-Refactor Cycle:**
  1. **Red**: Viết test trước (fail)
  2. **Green**: Viết code tối thiểu để pass test
  3. **Refactor**: Cải thiện code, giữ test pass

  ```js
  // 1. Red - Write failing test first
  test('calculate total price with tax', () => {
    expect(calculateTotal(100, 0.1)).toBe(110);
  });

  // 2. Green - Implement minimal code
  function calculateTotal(price, taxRate) {
    return price + (price * taxRate);
  }

  // 3. Refactor - Improve code quality
  function calculateTotal(price, taxRate) {
    if (price < 0 || taxRate < 0) throw new Error('Invalid input');
    return Math.round((price * (1 + taxRate)) * 100) / 100;
  }
  ```

## Testing Best Practices
1. **AAA Pattern**: Arrange, Act, Assert
   ```js
   test('should add item to cart', () => {
     // Arrange
     const cart = new Cart();
     const item = { id: 1, name: 'Product', price: 100 };
     
     // Act
     cart.addItem(item);
     
     // Assert
     expect(cart.items).toHaveLength(1);
     expect(cart.total).toBe(100);
   });
   ```

2. **Test Naming**: Mô tả hành vi, không phải implementation
   ```js
   // Good
   test('should return 400 when email is missing')
   test('should display error message when login fails')
   
   // Bad  
   test('test login function')
   test('should call validateEmail')
   ```

3. **One Assertion Per Test**: Mỗi test chỉ test một behavior
4. **Test Data Independence**: Mỗi test độc lập, không phụ thuộc test khác
5. **Mock External Dependencies**: Database, API calls, file system

## Performance Testing Basics
- **Load Testing với Artillery:**
  ```yaml
  # load-test.yml
  config:
    target: 'http://localhost:3000'
    phases:
      - duration: 60
        arrivalRate: 10
  scenarios:
    - name: "API Load Test"
      requests:
        - get:
            url: "/api/users"
  ```

- **Memory Leak Testing:**
  ```js
  test('should not leak memory', () => {
    const initialMemory = process.memoryUsage().heapUsed;
    
    // Run operation many times
    for (let i = 0; i < 10000; i++) {
      processLargeData();
    }
    
    global.gc(); // Force garbage collection
    const finalMemory = process.memoryUsage().heapUsed;
    
    expect(finalMemory - initialMemory).toBeLessThan(10 * 1024 * 1024); // 10MB
  });
  ```

## Công cụ phổ biến
- **Unit/Integration**: Jest, Mocha + Chai, Vitest
- **React Testing**: React Testing Library, Enzyme (deprecated)
- **E2E**: Cypress, Playwright, Puppeteer
- **API Testing**: Supertest, Postman, Insomnia
- **Performance**: Artillery, k6, Apache Bench
- **Coverage**: Jest (built-in), NYC, Istanbul
