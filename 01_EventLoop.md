# 1. Event Loop
- Event Loop là cơ chế giúp Node.js xử lý nhiều tác vụ bất đồng bộ (I/O, network, timer, v.v.) mà không bị block, cho phép server phục vụ nhiều request cùng lúc trên một luồng duy nhất.
- Khi một tác vụ bất đồng bộ (như đọc file, truy vấn DB) được gọi, Node.js sẽ đăng ký callback và tiếp tục xử lý các tác vụ khác, khi tác vụ hoàn thành, callback sẽ được đưa vào hàng đợi để thực thi.
- Các giai đoạn (phases) chính của Event Loop:
  1. **timers**: Xử lý các callback của setTimeout, setInterval.
  2. **pending callbacks**: Xử lý các I/O callback bị hoãn lại từ vòng lặp trước.
  3. **idle/prepare**: Dùng nội bộ cho Node.js, ít khi can thiệp.
  4. **poll**: Chờ các I/O events mới, thực thi callback liên quan đến I/O (đọc file, network, ...).
  5. **check**: Xử lý các callback của setImmediate.
  6. **close callbacks**: Xử lý các callback khi đóng connection (vd: socket.on('close')).
- Mỗi vòng lặp (tick) sẽ đi qua các phase này theo thứ tự trên. Nếu phase nào có callback thì sẽ thực thi, nếu không sẽ chuyển sang phase tiếp theo.
- Tìm hiểu thêm: https://nodejs.dev/en/learn/the-nodejs-event-loop/
