# 9. Performance & Memory
- **Phát hiện memory leak**:
  - Dấu hiệu: app chậm dần, RAM tăng liên tục, process bị kill do hết bộ nhớ.
  - Dùng `process.memoryUsage()` để theo dõi bộ nhớ:
    ```js
    setInterval(() => {
      console.log(process.memoryUsage());
    }, 5000);
    ```
  - Dùng package `heapdump` để chụp snapshot heap, phân tích bằng Chrome DevTools:
    ```js
    const heapdump = require('heapdump');
    heapdump.writeSnapshot('heap-' + Date.now() + '.heapsnapshot');
    ```
    - Mở file .heapsnapshot bằng Chrome DevTools (tab Memory) để tìm object bị giữ lại bất thường.

- **Profiling/tối ưu hiệu năng**:
  - Dùng `--inspect` hoặc `--inspect-brk` để debug/profiling với Chrome DevTools:
    ```sh
    node --inspect index.js
    ```
  - Dùng module `v8` để lấy thông tin heap statistics:
    ```js
    const v8 = require('v8');
    console.log(v8.getHeapStatistics());
    ```
  - Dùng các tool như clinic.js, autocannon để benchmark, phát hiện bottleneck.

- **Best practices**:
  - Giải phóng resource không dùng nữa (close DB, file, socket).
  - Tránh giữ reference tới object không cần thiết (global, closure).
  - Sử dụng stream thay vì load toàn bộ file/data lớn vào RAM.
  - Theo dõi log, alert khi memory tăng bất thường.
