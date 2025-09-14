# 14. TypeScript
- **Lợi ích:**
  - Phát hiện lỗi khi biên dịch (compile-time), giảm bug runtime.
  - Tự động gợi ý code, refactor dễ dàng, code rõ ràng hơn nhờ typing.
  - Dễ bảo trì, mở rộng dự án lớn.

- **Cấu hình cơ bản:**
  - Cài đặt: `npm install typescript @types/node --save-dev`
  - Tạo file cấu hình: `npx tsc --init`
  - Biên dịch: `npx tsc` (biến .ts thành .js)
  - Thêm script vào package.json:
    ```json
    "scripts": {
      "build": "tsc",
      "start": "node dist/index.js"
    }
    ```

- **Typing cho Node.js:**
  - Sử dụng các kiểu dữ liệu cơ bản: string, number, boolean, array, object, ...
  - Định nghĩa type cho biến, function, class:
    ```ts
    function sum(a: number, b: number): number {
      return a + b;
    }
    ```
  - Sử dụng type/interface để mô tả cấu trúc object:
    ```ts
    interface User {
      id: number;
      name: string;
    }
    const user: User = { id: 1, name: 'A' };
    ```
  - Cài đặt @types cho các thư viện bên ngoài (vd: `npm install @types/express`)
