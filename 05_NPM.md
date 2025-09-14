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

- **Semantic Versioning (Semver)**:
  - Format: `MAJOR.MINOR.PATCH` (vd: 1.2.3)
  - `^1.2.3`: Chấp nhận updates MINOR và PATCH (>= 1.2.3 < 2.0.0)
  - `~1.2.3`: Chỉ chấp nhận updates PATCH (>= 1.2.3 < 1.3.0)
  - `1.2.3`: Version cố định
  - `*` hoặc `latest`: Version mới nhất

- **package-lock.json**:
  - Lock exact versions của dependencies và sub-dependencies
  - Đảm bảo consistent installs across environments
  - Nên commit vào git để team có cùng versions

- **NPM Scripts nâng cao**:
  ```json
  {
    "scripts": {
      "start": "node server.js",
      "dev": "nodemon server.js",
      "build": "webpack --mode=production",
      "test": "jest",
      "test:watch": "jest --watch",
      "prestart": "npm run build",
      "postinstall": "node setup.js",
      "clean": "rm -rf dist"
    }
  }
  ```
  - `pre` và `post` hooks tự động chạy trước/sau script chính
  - `npm run` để chạy custom scripts

- **NPM Config & Registry**:
  ```bash
  # Xem config hiện tại
  npm config list
  
  # Set registry (cho private packages)
  npm config set registry https://registry.company.com
  
  # Set proxy
  npm config set proxy http://proxy.company.com:8080
  
  # Đăng nhập npm
  npm login
  
  # Publish package
  npm publish
  ```

- **Local vs Global Packages**:
  ```bash
  # Local install (trong node_modules của project)
  npm install express
  
  # Global install (system-wide)
  npm install -g nodemon
  
  # Chạy package local thông qua npx
  npx create-react-app my-app
  npx jest
  ```

- **NPX (NPM Package Runner)**:
  - Chạy packages mà không cần cài global
  - Luôn dùng latest version
  - Ví dụ: `npx create-react-app`, `npx eslint`

- **Workspaces** (NPM 7+):
  ```json
  {
    "name": "my-project",
    "workspaces": [
      "packages/*",
      "apps/*"
    ]
  }
  ```
  - Quản lý monorepo với nhiều packages
  - Shared dependencies, symlinked packages

- **Scoped Packages**:
  ```bash
  # Install scoped package
  npm install @angular/core
  npm install @company/shared-utils
  
  # Publish scoped package
  npm publish --access public
  ```

- **Environment Variables trong NPM**:
  ```json
  {
    "scripts": {
      "start:dev": "NODE_ENV=development node app.js",
      "start:prod": "NODE_ENV=production node app.js",
      "build": "NODE_ENV=production webpack"
    }
  }
  ```

- **NPM Cache**:
  ```bash
  # Xem cache location
  npm config get cache
  
  # Clear cache
  npm cache clean --force
  
  # Verify cache
  npm cache verify
  ```

## Câu hỏi phỏng vấn thường gặp:

### 1. **package.json vs package-lock.json**
- Q: Sự khác biệt và tại sao cần package-lock.json?
- A: package.json định nghĩa dependencies với version ranges, package-lock.json lock exact versions. Đảm bảo consistent installs.

### 2. **Semantic Versioning**
- Q: `^1.2.3` và `~1.2.3` khác nhau như thế nào?
- A: `^` cho phép minor/patch updates, `~` chỉ cho phép patch updates. `^` có thể breaking changes.

### 3. **dependencies vs devDependencies**
- Q: Khi nào dùng devDependencies?
- A: devDependencies cho tools development (testing, building, linting). Không được install trong production (`npm install --production`).

### 4. **NPM vs Yarn vs PNPM**
- Q: So sánh các package managers?
- A: 
  - NPM: Default, stable, workspace support
  - Yarn: Faster, better caching, workspaces
  - PNPM: Disk efficient, symlink approach

### 5. **Security Issues**
- Q: Làm sao check security vulnerabilities?
- A: `npm audit` để scan, `npm audit fix` để tự động fix. Dùng `npm audit --audit-level=high` cho strict check.

### 6. **Monorepo Management**
- Q: Quản lý monorepo với NPM workspaces?
- A: Dùng `workspaces` field trong package.json, `npm install -w workspace-name`, shared dependencies.

### 7. **Performance Optimization**
- Q: Làm sao optimize npm install speed?
- A: 
  - Dùng npm ci trong CI/CD
  - Cache node_modules
  - Dùng .npmrc với registry gần
  - Parallel installs với --maxsockets

### 8. **Private Packages**
- Q: Deploy private npm packages như thế nào?
- A: Dùng private registry (Verdaccio, NPM Enterprise), scoped packages với authentication.
