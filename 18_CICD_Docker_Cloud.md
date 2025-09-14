# 18. CI/CD, Docker, Cloud
- **Dockerfile**: Đóng gói app Node.js thành image, dễ deploy, scale.
  - Ví dụ:
    ```Dockerfile
    FROM node:18-alpine
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    EXPOSE 3000
    CMD ["node", "index.js"]
    ```

- **docker-compose**: Quản lý nhiều service (app, db, redis, ...), dễ chạy local/dev.
  - Ví dụ:
    ```yaml
    version: '3'
    services:
      app:
        build: .
        ports:
          - "3000:3000"
        environment:
          - NODE_ENV=production
      db:
        image: mongo
        ports:
          - "27017:27017"
    ```

- **Pipeline CI/CD**: Tự động build, test, deploy khi push code (dùng Github Actions, Gitlab CI, Jenkins, ...).
  - Ví dụ Github Actions:
    ```yaml
    name: CI
    on: [push]
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v2
          - uses: actions/setup-node@v3
            with:
              node-version: 18
          - run: npm install
          - run: npm test
          - run: docker build -t myapp .
    ```

- **Deploy lên cloud**:
  - Có thể deploy image lên các dịch vụ như AWS ECS, Google Cloud Run, Azure Container Apps, Heroku, Render, ...
  - Thường chỉ cần push image lên registry (Docker Hub, ECR, GCR), cấu hình service và scale.
