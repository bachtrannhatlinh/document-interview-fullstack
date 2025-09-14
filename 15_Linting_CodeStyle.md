# 15. Linting & Code Style
- **ESLint**: Kiểm tra, enforce quy tắc code (style, bug, best practice) cho JS/TS.
  - Cài đặt: `npm install eslint --save-dev`
  - Khởi tạo: `npx eslint --init`
  - Chạy lint: `npx eslint .`

- **Prettier**: Định dạng code tự động, nhất quán style (dấu cách, dấu phẩy, xuống dòng, ...).
  - Cài đặt: `npm install prettier --save-dev`
  - Tích hợp với ESLint: `npm install eslint-config-prettier eslint-plugin-prettier --save-dev`
  - Chạy format: `npx prettier --write .`

- **husky**: Tạo git hook tự động (chạy lint/test trước khi commit/push).
  - Cài đặt: `npm install husky --save-dev`
  - Khởi tạo: `npx husky install`
  - Thêm hook: `npx husky add .husky/pre-commit "npm run lint"`

- **commitlint**: Kiểm tra message commit theo chuẩn (vd: Conventional Commit).
  - Cài đặt: `npm install @commitlint/{config-conventional,cli} --save-dev`
  - Tạo file cấu hình commitlint.config.js:
    ```js
    module.exports = { extends: ['@commitlint/config-conventional'] };
    ```
  - Thêm hook với husky: `npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"`
