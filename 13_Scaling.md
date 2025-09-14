# 13. Scaling
- **Cluster**: Node.js chạy đơn luồng, muốn tận dụng đa nhân CPU thì dùng module `cluster` để tạo nhiều process con (worker) chia sẻ port, tăng throughput.
  - Ví dụ:
    ```js
    const cluster = require('cluster');
    const os = require('os');
    if (cluster.isMaster) {
      for (let i = 0; i < os.cpus().length; i++) {
        cluster.fork();
      }
    } else {
      // worker code
      require('./app');
    }
    ```

- **child_process**: Tạo process con để chạy task nặng, tách biệt với main process (spawn, fork, exec, ...).
  - Ví dụ:
    ```js
    const { spawn } = require('child_process');
    const ls = spawn('ls', ['-lh', '/usr']);
    ls.stdout.on('data', data => console.log(`stdout: ${data}`));
    ```

- **worker_threads**: Tạo thread con để xử lý tính toán nặng mà không block event loop (Node.js >= v10.5).
  - Ví dụ:
    ```js
    const { Worker } = require('worker_threads');
    new Worker('./worker.js');
    ```

- **Load balancing**: Phân phối request đều cho nhiều instance Node.js (dùng cluster, hoặc deploy nhiều container, dùng Nginx/HAProxy làm load balancer).
  - Thường dùng kèm Docker/Kubernetes để scale ngang.
