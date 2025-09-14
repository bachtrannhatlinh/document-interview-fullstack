
> Tài liệu đã được tách thành các file nhỏ, mỗi file cho một chủ đề. Click vào từng mục để xem chi tiết.

1. [Event Loop](./01_EventLoop.md)
2. [Callback, Promise, Async/Await](./02_Callback_Promise_AsyncAwait.md)
3. [Module System](./03_ModuleSystem.md)
4. [Built-in Modules](./04_BuiltInModules.md)
5. [Quản lý package với NPM](./05_NPM.md)
6. [Xử lý lỗi](./06_ErrorHandling.md)
7. [Debug & Logging](./07_Debug_Logging.md)
8. [Best Practices](./08_BestPractices.md)
9. [Performance & Memory](./09_Performance_Memory.md)
10. [Security](./10_Security.md)
11. [Testing](./11_Testing.md)
12. [Deployment & Monitoring](./12_Deployment_Monitoring.md)
13. [Scaling](./13_Scaling.md)
14. [TypeScript](./14_TypeScript.md)
15. [Linting & Code Style](./15_Linting_CodeStyle.md)
16. [Express/NestJS](./16_Express_NestJS.md)
17. [Database & REST API](./17_Database_RESTAPI.md)
18. [CI/CD, Docker, Cloud](./18_CICD_Docker_Cloud.md)
19. [Tài liệu tham khảo](./19_TaiLieu_ThamKhao.md)

> Tài liệu đã được tách thành các file nhỏ, mỗi file cho một chủ đề. Click vào từng mục để xem chi tiết.

1. [Event Loop](./01_EventLoop.md)
2. [Callback, Promise, Async/Await](./02_Callback_Promise_AsyncAwait.md)
3. [Module System](./03_ModuleSystem.md)
4. [Built-in Modules](./04_BuiltInModules.md)
5. [Quản lý package với NPM](./05_NPM.md)
6. [Xử lý lỗi](./06_ErrorHandling.md)
7. [Debug & Logging](./07_Debug_Logging.md)
8. [Best Practices](./08_BestPractices.md)
9. [Performance & Memory](./09_Performance_Memory.md)
10. [Security](./10_Security.md)
11. [Testing](./11_Testing.md)
12. [Deployment & Monitoring](./12_Deployment_Monitoring.md)
13. [Scaling](./13_Scaling.md)
14. [TypeScript](./14_TypeScript.md)
15. [Linting & Code Style](./15_Linting_CodeStyle.md)
16. [Express/NestJS](./16_Express_NestJS.md)
17. [Database & REST API](./17_Database_RESTAPI.md)
18. [CI/CD, Docker, Cloud](./18_CICD_Docker_Cloud.md)
19. [Tài liệu tham khảo](./19_TaiLieu_ThamKhao.md)
## 17. Express/NestJS
- **Cấu trúc project**:
	- Express: Thường chia thành các folder như routes, controllers, services, middlewares, models, utils.
	- NestJS: Theo module (module, controller, service, dto, guard, ...), rõ ràng, dễ mở rộng.

- **Middleware**:
	- Express: Hàm xử lý request/response trước khi vào route handler.
		```js
		app.use((req, res, next) => { console.log(req.url); next(); });
		```
	- NestJS: Dùng class implements NestMiddleware, đăng ký trong module.

- **Error middleware**:
	- Express: Middleware nhận 4 tham số (err, req, res, next), dùng để xử lý lỗi tập trung.
		```js
		app.use((err, req, res, next) => { res.status(500).json({ error: err.message }); });
		```
	- NestJS: Dùng Exception Filter (implements ExceptionFilter), đăng ký toàn cục hoặc cho từng controller.

- **Validation**:
	- Express: Dùng express-validator, joi, yup để validate input.
		```js
		const { body, validationResult } = require('express-validator');
		app.post('/user', body('email').isEmail(), (req, res) => { ... });
		```
	- NestJS: Dùng class-validator, class-transformer, DTO để validate tự động.
		```ts
		import { IsEmail } from 'class-validator';
		class CreateUserDto { @IsEmail() email: string; }
		```

- **Authentication**:
	- Express: Dùng passport, jsonwebtoken để xác thực JWT, session, OAuth2.
		```js
		const jwt = require('jsonwebtoken');
		const token = jwt.sign({ id: user.id }, 'secret');
		```
	- NestJS: Dùng Passport module, guard, strategy (JWT, local, OAuth2, ...).
## 16. Linting & Code Style
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
## 15. TypeScript
- **Lợi ích:**
	- Phát hiện lỗi khi biên dịch (compile-time), giảm bug runtime.
	- Tự động gợi ý code, refactor dễ dàng, code rõ ràng hơn nhờ typing.
	- Dễ bảo trì, mở rộng dự án lớn.

- **Cấu hình cơ bản:**
	- Cài đặt: `npm install typescript @types/node --save-dev`
	- Tạo file cấu hình: `npx tsc --init`
	- Biên dịch: `npx tsc` (biến .ts thành .js)
	- Thêm script vào package.json:
		```json
		"scripts": {
			"build": "tsc",
			"start": "node dist/index.js"
		}
		```

- **Typing cho Node.js:**
	- Sử dụng các kiểu dữ liệu cơ bản: string, number, boolean, array, object, ...
	- Định nghĩa type cho biến, function, class:
		```ts
		function sum(a: number, b: number): number {
			return a + b;
		}
		```
	- Sử dụng type/interface để mô tả cấu trúc object:
		```ts
		interface User {
			id: number;
			name: string;
		}
		const user: User = { id: 1, name: 'A' };
		```
	- Cài đặt @types cho các thư viện bên ngoài (vd: `npm install @types/express`)
## 14. Scaling
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
## 13. Deployment & Monitoring
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
## 12. Testing
- **Unit test**: Kiểm thử từng function/module nhỏ, đảm bảo logic hoạt động đúng độc lập.
	- Ví dụ (Jest):
		```js
		// math.js
		function add(a, b) { return a + b; }
		module.exports = { add };
		// math.test.js
		const { add } = require('./math');
		test('add 2 + 3 = 5', () => {
			expect(add(2, 3)).toBe(5);
		});
		```

- **Integration test**: Kiểm thử nhiều module/service kết hợp, thường test API endpoint.
	- Ví dụ (Supertest + Jest):
		```js
		const request = require('supertest');
		const app = require('./app');
		test('GET /users trả về 200', async () => {
			const res = await request(app).get('/users');
			expect(res.statusCode).toBe(200);
		});
		```

- **Mock**: Giả lập function/module/phụ thuộc bên ngoài (DB, API, ...), giúp test độc lập, không phụ thuộc môi trường thật.
	- Ví dụ (Jest):
		```js
		jest.mock('./db');
		const db = require('./db');
		db.getUser.mockReturnValue({ id: 1, name: 'A' });
		```

- **Coverage**: Đo lường tỷ lệ code được test (statement, branch, function, line).
	- Chạy với Jest: `jest --coverage`

- **Công cụ phổ biến**:
	- **Jest**: Đơn giản, mạnh mẽ, hỗ trợ mock, coverage, phù hợp cả unit & integration test.
	- **Mocha**: Linh hoạt, kết hợp với Chai (assertion), Sinon (mock/stub).
	- **Supertest**: Test API endpoint cho Express/NestJS.
## 11. Security
- **Các lỗ hổng phổ biến:**
	- **XSS (Cross-Site Scripting):** Chèn mã độc vào web, thường gặp khi render dữ liệu không kiểm soát lên client. Phòng tránh: escape output, không render trực tiếp dữ liệu user nhập.
	- **CSRF (Cross-Site Request Forgery):** Kẻ tấn công lợi dụng session của user để gửi request giả mạo. Phòng tránh: dùng CSRF token, kiểm tra Origin/Referer.
	- **SQL Injection:** Chèn mã SQL vào query, thường gặp khi dùng query string trực tiếp với DB. Phòng tránh: dùng ORM, prepared statement, không nối chuỗi query.
	- **NoSQL Injection:** Tương tự SQLi nhưng với NoSQL (MongoDB, ...), ví dụ truyền object `{ "$gt": "" }` để bypass auth. Phòng tránh: validate input, không truyền object trực tiếp vào query.

- **Cách phòng tránh:**
	- **helmet:** Middleware bảo vệ HTTP headers cho Express/NestJS.
		```js
		const helmet = require('helmet');
		app.use(helmet());
		```
	- **sanitize:** Làm sạch input, loại bỏ ký tự nguy hiểm (dùng thư viện như express-validator, validator, mongo-sanitize).
		```js
		const { sanitize } = require('express-validator');
		app.post('/user', sanitize('name').escape(), ...);
		```
	- **validate input:** Kiểm tra dữ liệu đầu vào (type, length, format, ...), reject nếu không hợp lệ.
		```js
		const { body, validationResult } = require('express-validator');
		app.post('/user', body('email').isEmail(), (req, res) => {
			const errors = validationResult(req);
			if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
			// ...
		});
		```
	- Không trả lỗi chi tiết cho client, log lỗi ở server.
	- Cập nhật dependencies thường xuyên, kiểm tra lỗ hổng với `npm audit`.
## 10. Performance & Memory
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
# Node.js Chi Tiết

## 1. Event Loop
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

## 2. Callback, Promise, Async/Await
- **Callback**: Là hàm được truyền vào một hàm khác như một đối số, sẽ được gọi lại (callback) khi tác vụ bất đồng bộ hoàn thành. Dễ gặp callback hell khi lồng nhiều callback.
  
	Ví dụ:
	```js
	fs.readFile('file.txt', 'utf8', (err, data) => {
		if (err) throw err;
		console.log(data);
	});
	```

- **Promise**: Là một đối tượng đại diện cho một giá trị có thể có trong tương lai (thành công hoặc thất bại). Giúp tránh callback hell, dễ quản lý chuỗi tác vụ bất đồng bộ.
  
	Ví dụ:
	```js
	function readFilePromise(path) {
		return new Promise((resolve, reject) => {
			fs.readFile(path, 'utf8', (err, data) => {
				if (err) reject(err);
				else resolve(data);
			});
		});
	}
	readFilePromise('file.txt')
		.then(data => console.log(data))
		.catch(err => console.error(err));
	```

- **Async/Await**: Cú pháp giúp viết code bất đồng bộ trông như đồng bộ, dễ đọc, dễ debug hơn. Dựa trên Promise.
  
	Ví dụ:
	```js
	async function readFileAsync() {
		try {
			const data = await readFilePromise('file.txt');
			console.log(data);
		} catch (err) {
			console.error(err);
		}
	}
	readFileAsync();
	```

## 3. Module System
- **CommonJS** (chuẩn cũ, phổ biến nhất):
	- Sử dụng `require()` để import module, `module.exports` để export.
	- Mặc định dùng trong Node.js trước v14.
	- Ví dụ:
		```js
		// math.js
		function add(a, b) { return a + b; }
		module.exports = { add };
		// app.js
		const math = require('./math');
		console.log(math.add(2, 3));
		```

- **ES Module** (ECMAScript Module, chuẩn mới):
	- Sử dụng `import`/`export` giống như trên trình duyệt.
	- Hỗ trợ tốt từ Node.js v14+ (cần đặt "type": "module" trong package.json hoặc dùng đuôi .mjs).
	- Ví dụ:
		```js
		// math.mjs
		export function add(a, b) { return a + b; }
		// app.mjs
		import { add } from './math.mjs';
		console.log(add(2, 3));
		```

- **Tự tạo module**: Chia nhỏ code thành nhiều file/module để dễ bảo trì, tái sử dụng.

- **Sử dụng built-in module**: Node.js cung cấp nhiều module sẵn như `fs`, `path`, `http`, ...
	- Ví dụ:
		```js
		const fs = require('fs');
		fs.readFile('file.txt', 'utf8', (err, data) => {
			if (err) throw err;
			console.log(data);
		});
		```

## 4. Built-in Modules
- **`fs` (File System)**: Đọc/ghi file, thao tác với hệ thống file.
	- Ví dụ:
		```js
		const fs = require('fs');
		fs.readFile('file.txt', 'utf8', (err, data) => {
			if (err) throw err;
			console.log(data);
		});
		```

- **`http`**: Tạo server HTTP, xây dựng web server cơ bản.
	- Ví dụ:
		```js
		const http = require('http');
		const server = http.createServer((req, res) => {
			res.end('Hello Node.js');
		});
		server.listen(3000);
		```

- **`path`**: Xử lý đường dẫn file, join, resolve, basename, extname...
	- Ví dụ:
		```js
		const path = require('path');
		const filePath = path.join(__dirname, 'file.txt');
		console.log(filePath);
		```

- **`os`**: Lấy thông tin hệ điều hành (CPU, memory, platform, ...).
	- Ví dụ:
		```js
		const os = require('os');
		console.log(os.platform(), os.cpus().length);
		```

- **`events`**: Tạo và quản lý các sự kiện (EventEmitter pattern).
	- Ví dụ:
		```js
		const EventEmitter = require('events');
		const emitter = new EventEmitter();
		emitter.on('greet', name => console.log('Hello', name));
		emitter.emit('greet', 'Node.js');
		```

- **`stream`**: Xử lý dữ liệu lớn theo luồng (streaming), đọc/ghi file lớn, network...
	- Ví dụ:
		```js
		const fs = require('fs');
		const readStream = fs.createReadStream('bigfile.txt');
		readStream.on('data', chunk => {
			console.log('Received:', chunk.length, 'bytes');
		});
		```

## 5. Quản lý package với NPM
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

## 6. Xử lý lỗi
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

## 7. Debug & Logging
- Sử dụng `console.log`, `console.error`
- Debug với VSCode, Chrome DevTools
- Thư viện logging: winston, morgan

## 8. Best Practices
- **Không dùng code đồng bộ (blocking) trong production**: Tránh dùng các hàm sync như `fs.readFileSync`, vì sẽ chặn event loop, làm giảm hiệu năng server.
	- Ví dụ nên tránh:
		```js
		// Không nên dùng
		const data = fs.readFileSync('file.txt', 'utf8');
		```
	- Thay vào đó, hãy dùng async:
		```js
		fs.readFile('file.txt', 'utf8', (err, data) => { ... });
		```

- **Tách nhỏ module, dễ test, dễ bảo trì**: Chia code thành nhiều file/module nhỏ (service, controller, utils, ...), mỗi module chỉ làm một nhiệm vụ (Single Responsibility Principle).
	- Giúp code dễ đọc, dễ test, dễ mở rộng.

- **Sử dụng environment variable cho config**: Không hard-code thông tin nhạy cảm (DB, API key, ...), dùng biến môi trường và thư viện như dotenv.
	- Ví dụ:
		```js
		require('dotenv').config();
		const dbUrl = process.env.DB_URL;
		```

- **Xử lý lỗi đầy đủ, không để app crash**: Luôn bắt lỗi (try/catch, callback, global handler), log lỗi, trả response hợp lý cho client, không để app dừng đột ngột.
	- Có thể dùng các tool giám sát như PM2, log lỗi gửi về monitoring system.

## 9. Tài liệu tham khảo
- https://nodejs.org/en/docs
- https://nodejs.dev/en/learn/
- https://www.freecodecamp.org/news/the-definitive-node-js-handbook-6912378afc6e/
