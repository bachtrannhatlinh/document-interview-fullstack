# 8. Best Practices
- **Không dùng code đồng bộ (blocking) trong production**: Tránh dùng các hàm sync như `fs.readFileSync`, vì sẽ chặn event loop, làm giảm hiệu năng server.
  - Ví dụ nên tránh:
    ```js
    // Không nên dùng
    const data = fs.readFileSync('file.txt', 'utf8');
    ```
  - Thay vào đó, hãy dùng async:
    ```js
    fs.readFile('file.txt', 'utf8', (err, data) => { ... });
    ```

- **Tách nhỏ module, dễ test, dễ bảo trì**: Chia code thành nhiều file/module nhỏ (service, controller, utils, ...), mỗi module chỉ làm một nhiệm vụ (Single Responsibility Principle).
  - Giúp code dễ đọc, dễ test, dễ mở rộng.

- **Sử dụng environment variable cho config**: Không hard-code thông tin nhạy cảm (DB, API key, ...), dùng biến môi trường và thư viện như dotenv.
  - Ví dụ:
    ```js
    require('dotenv').config();
    const dbUrl = process.env.DB_URL;
    ```

- **Xử lý lỗi đầy đủ, không để app crash**: Luôn bắt lỗi (try/catch, callback, global handler), log lỗi, trả response hợp lý cho client, không để app dừng đột ngột.
  - Có thể dùng các tool giám sát như PM2, log lỗi gửi về monitoring system.
