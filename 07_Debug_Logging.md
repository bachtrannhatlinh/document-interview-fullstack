# 7. Debug & Logging
- Sử dụng `console.log`, `console.error` để debug đơn giản.
- Debug với VSCode, Chrome DevTools (node --inspect, breakpoint, ...).
- Thư viện logging chuyên nghiệp: winston, morgan, pino.
  - Ví dụ winston:
    ```js
    const winston = require('winston');
    const logger = winston.createLogger({
      transports: [new winston.transports.File({ filename: 'app.log' })]
    });
    logger.info('App started');
    ```
