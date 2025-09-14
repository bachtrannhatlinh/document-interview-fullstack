# 3. Module System
- **CommonJS** (chuẩn cũ, phổ biến nhất):
  - Sử dụng `require()` để import module, `module.exports` để export.
  - Mặc định dùng trong Node.js trước v14.
  - Ví dụ:
    ```js
    // math.js
    function add(a, b) { return a + b; }
    module.exports = { add };
    // app.js
    const math = require('./math');
    console.log(math.add(2, 3));
    ```

- **ES Module** (ECMAScript Module, chuẩn mới):
  - Sử dụng `import`/`export` giống như trên trình duyệt.
  - Hỗ trợ tốt từ Node.js v14+ (cần đặt "type": "module" trong package.json hoặc dùng đuôi .mjs).
  - Ví dụ:
    ```js
    // math.mjs
    export function add(a, b) { return a + b; }
    // app.mjs
    import { add } from './math.mjs';
    console.log(add(2, 3));
    ```

- **Tự tạo module**: Chia nhỏ code thành nhiều file/module để dễ bảo trì, tái sử dụng.

- **Sử dụng built-in module**: Node.js cung cấp nhiều module sẵn như `fs`, `path`, `http`, ...
  - Ví dụ:
    ```js
    const fs = require('fs');
    fs.readFile('file.txt', 'utf8', (err, data) => {
      if (err) throw err;
      console.log(data);
    });
    ```
