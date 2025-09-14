# 5. Quản lý package với NPM
- **NPM (Node Package Manager)** là công cụ quản lý thư viện/phụ thuộc cho Node.js.

- **Các lệnh npm cơ bản:**
  - `npm init`: Khởi tạo project Node.js mới, tạo file package.json.
  - `npm install <package>`: Cài đặt package vào project (mặc định vào dependencies).
  - `npm install <package> --save-dev`: Cài đặt package cho môi trường dev (devDependencies).
  - `npm uninstall <package>`: Gỡ bỏ package khỏi project.
  - `npm update`: Cập nhật các package lên phiên bản mới nhất phù hợp với package.json.
  - `npm install`: Cài đặt tất cả package có trong package.json.

- **package.json**:
  - Lưu thông tin project, dependencies, scripts, version, author, ...
  - Ví dụ:
    ```json
    {
      "name": "my-app",
      "version": "1.0.0",
      "main": "index.js",
      "scripts": {
        "start": "node index.js",
        "test": "jest"
      },
      "dependencies": {
        "express": "^4.18.0"
      },
      "devDependencies": {
        "jest": "^29.0.0"
      }
    }
    ```
  - dependencies: Thư viện dùng cho production.
  - devDependencies: Thư viện chỉ dùng cho phát triển/test.
  - scripts: Định nghĩa các lệnh tiện ích (start, test, build, ...).
