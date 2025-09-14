# 10. Security
- **Các lỗ hổng phổ biến:**
  - **XSS (Cross-Site Scripting):** Chèn mã độc vào web, thường gặp khi render dữ liệu không kiểm soát lên client. Phòng tránh: escape output, không render trực tiếp dữ liệu user nhập.
  - **CSRF (Cross-Site Request Forgery):** Kẻ tấn công lợi dụng session của user để gửi request giả mạo. Phòng tránh: dùng CSRF token, kiểm tra Origin/Referer.
  - **SQL Injection:** Chèn mã SQL vào query, thường gặp khi dùng query string trực tiếp với DB. Phòng tránh: dùng ORM, prepared statement, không nối chuỗi query.
  - **NoSQL Injection:** Tương tự SQLi nhưng với NoSQL (MongoDB, ...), ví dụ truyền object `{ "$gt": "" }` để bypass auth. Phòng tránh: validate input, không truyền object trực tiếp vào query.

- **Cách phòng tránh:**
  - **helmet:** Middleware bảo vệ HTTP headers cho Express/NestJS.
    ```js
    const helmet = require('helmet');
    app.use(helmet());
    ```
  - **sanitize:** Làm sạch input, loại bỏ ký tự nguy hiểm (dùng thư viện như express-validator, validator, mongo-sanitize).
    ```js
    const { sanitize } = require('express-validator');
    app.post('/user', sanitize('name').escape(), ...);
    ```
  - **validate input:** Kiểm tra dữ liệu đầu vào (type, length, format, ...), reject nếu không hợp lệ.
    ```js
    const { body, validationResult } = require('express-validator');
    app.post('/user', body('email').isEmail(), (req, res) => {
      const errors = validationResult(req);
      if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
      // ...
    });
    ```
  - Không trả lỗi chi tiết cho client, log lỗi ở server.
  - Cập nhật dependencies thường xuyên, kiểm tra lỗ hổng với `npm audit`.
