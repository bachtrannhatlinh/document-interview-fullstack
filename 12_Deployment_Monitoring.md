# 12. Deployment & Monitoring
- **PM2**: Quản lý process Node.js, tự restart khi crash, scale nhiều instance, log, monitoring.
  - Cài đặt: `npm install -g pm2`
  - Chạy app: `pm2 start index.js`
  - Xem log: `pm2 logs`
  - Scale: `pm2 scale index 4` (chạy 4 instance)
  - Tự động restart khi server reboot: `pm2 startup`

- **Log**: Ghi log ra file hoặc console để theo dõi hoạt động, lỗi (dùng winston, morgan, ...).
  - Ví dụ dùng winston:
    ```js
    const winston = require('winston');
    const logger = winston.createLogger({
      transports: [new winston.transports.File({ filename: 'app.log' })]
    });
    logger.info('App started');
    ```

- **Healthcheck**: Tạo endpoint `/health` trả về trạng thái app (OK, DB connect, ...), giúp load balancer/monitor kiểm tra app còn sống.
  - Ví dụ:
    ```js
    app.get('/health', (req, res) => res.send('OK'));
    ```

- **Monitoring**:
  - **NewRelic**: Theo dõi hiệu năng, alert khi có vấn đề (cần đăng ký tài khoản, cài agent).
  - **Sentry**: Theo dõi lỗi runtime, gửi alert khi có exception.
  - **Prometheus**: Thu thập metrics (CPU, RAM, request, ...), kết hợp với Grafana để vẽ dashboard.
  - Có thể dùng các exporter cho Node.js như prom-client để expose metrics cho Prometheus.
