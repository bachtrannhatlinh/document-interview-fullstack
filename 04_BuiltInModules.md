# 4. Built-in Modules
- **fs (File System)**: Đọc/ghi file, thao tác với hệ thống file.
  - Ví dụ:
    ```js
    const fs = require('fs');
    fs.readFile('file.txt', 'utf8', (err, data) => {
      if (err) throw err;
      console.log(data);
    });
    ```

- **http**: Tạo server HTTP, xây dựng web server cơ bản.
  - Ví dụ:
    ```js
    const http = require('http');
    const server = http.createServer((req, res) => {
      res.end('Hello Node.js');
    });
    server.listen(3000);
    ```

- **path**: Xử lý đường dẫn file, join, resolve, basename, extname...
  - Ví dụ:
    ```js
    const path = require('path');
    const filePath = path.join(__dirname, 'file.txt');
    console.log(filePath);
    ```

- **os**: Lấy thông tin hệ điều hành (CPU, memory, platform, ...).
  - Ví dụ:
    ```js
    const os = require('os');
    console.log(os.platform(), os.cpus().length);
    ```

- **events**: Tạo và quản lý các sự kiện (EventEmitter pattern).
  - Ví dụ:
    ```js
    const EventEmitter = require('events');
    const emitter = new EventEmitter();
    emitter.on('greet', name => console.log('Hello', name));
    emitter.emit('greet', 'Node.js');
    ```

- **stream**: Xử lý dữ liệu lớn theo luồng (streaming), đọc/ghi file lớn, network...
  - Ví dụ:
    ```js
    const fs = require('fs');
    const readStream = fs.createReadStream('bigfile.txt');
    readStream.on('data', chunk => {
      console.log('Received:', chunk.length, 'bytes');
    });
    ```
