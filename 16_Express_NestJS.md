# 16. Express/NestJS
- **Cấu trúc project**:
  - Express: Thường chia thành các folder như routes, controllers, services, middlewares, models, utils.
  - NestJS: Theo module (module, controller, service, dto, guard, ...), rõ ràng, dễ mở rộng.

- **Middleware**:
  - Express: Hàm xử lý request/response trước khi vào route handler.
    ```js
    app.use((req, res, next) => { console.log(req.url); next(); });
    ```
  - NestJS: Dùng class implements NestMiddleware, đăng ký trong module.

- **Error middleware**:
  - Express: Middleware nhận 4 tham số (err, req, res, next), dùng để xử lý lỗi tập trung.
    ```js
    app.use((err, req, res, next) => { res.status(500).json({ error: err.message }); });
    ```
  - NestJS: Dùng Exception Filter (implements ExceptionFilter), đăng ký toàn cục hoặc cho từng controller.

- **Validation**:
  - Express: Dùng express-validator, joi, yup để validate input.
    ```js
    const { body, validationResult } = require('express-validator');
    app.post('/user', body('email').isEmail(), (req, res) => { ... });
    ```
  - NestJS: Dùng class-validator, class-transformer, DTO để validate tự động.
    ```ts
    import { IsEmail } from 'class-validator';
    class CreateUserDto { @IsEmail() email: string; }
    ```

- **Authentication**:
  - Express: Dùng passport, jsonwebtoken để xác thực JWT, session, OAuth2.
    ```js
    const jwt = require('jsonwebtoken');
    const token = jwt.sign({ id: user.id }, 'secret');
    ```
  - NestJS: Dùng Passport module, guard, strategy (JWT, local, OAuth2, ...).
