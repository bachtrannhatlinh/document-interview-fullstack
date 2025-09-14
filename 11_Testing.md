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

- **Công cụ phổ biến**:
  - **Jest**: Đơn giản, mạnh mẽ, hỗ trợ mock, coverage, phù hợp cả unit & integration test.
  - **Mocha**: Linh hoạt, kết hợp với Chai (assertion), Sinon (mock/stub).
  - **Supertest**: Test API endpoint cho Express/NestJS.
