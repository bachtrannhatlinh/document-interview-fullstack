# 6. Xử lý lỗi
- **try/catch với async/await**: Dùng để bắt lỗi khi sử dụng async/await với Promise.
  - Ví dụ:
    ```js
    async function readFileAsync() {
      try {
        const data = await readFilePromise('file.txt');
        console.log(data);
      } catch (err) {
        console.error('Lỗi:', err);
      }
    }
    ```

- **Xử lý lỗi callback (err-first callback)**: Theo convention của Node.js, callback luôn nhận tham số đầu tiên là error (nếu có lỗi), tiếp theo là kết quả.
  - Ví dụ:
    ```js
    fs.readFile('file.txt', 'utf8', (err, data) => {
      if (err) {
        console.error('Lỗi:', err);
        return;
      }
      console.log(data);
    });
    ```

- **Global error handler**: Bắt các lỗi không được xử lý ở cấp toàn cục để tránh app bị crash đột ngột.
  - Ví dụ:
    ```js
    process.on('uncaughtException', (err) => {
      console.error('Uncaught Exception:', err);
      // Có thể log lỗi, gửi alert, hoặc shutdown app an toàn
    });
    process.on('unhandledRejection', (reason, promise) => {
      console.error('Unhandled Rejection:', reason);
    });
    ```
