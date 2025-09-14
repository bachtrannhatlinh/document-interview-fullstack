# 16. Express/NestJS

## Express.js

### Khái niệm cơ bản
- **Express.js**: Web framework tối thiểu và linh hoạt cho Node.js
- **Unopinionated**: Không ép buộc cấu trúc, cho phép tự do thiết kế
- **Middleware-based**: Xây dựng dựa trên middleware pattern

### Cấu trúc project Express
```
project/
├── routes/          # Định tuyến
├── controllers/     # Logic xử lý
├── middlewares/     # Middleware tùy chỉnh
├── models/          # Models (database)
├── services/        # Business logic
├── utils/           # Utilities
├── config/          # Cấu hình
└── app.js           # Entry point
```

### Express Middleware
- **Middleware**: Hàm có quyền truy cập req, res, next
- **Thứ tự thực thi**: Theo thứ tự khai báo
```js
// Application-level middleware
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});

// Router-level middleware
const router = express.Router();
router.use('/user/:id', (req, res, next) => {
  console.log('Request Type:', req.method);
  next();
});

// Error-handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

### Express Routing
```js
// Basic routing
app.get('/', (req, res) => res.send('Hello World'));
app.post('/user', (req, res) => res.json({ user: 'created' }));

// Route parameters
app.get('/users/:id', (req, res) => {
  const id = req.params.id;
  res.json({ userId: id });
});

// Query parameters
app.get('/search', (req, res) => {
  const query = req.query.q;
  res.json({ searchTerm: query });
});

// Route handlers
app.get('/example', middleware1, middleware2, (req, res) => {
  res.send('Multiple handlers');
});
```

## NestJS

### Khái niệm cơ bản
- **NestJS**: Progressive Node.js framework
- **Opinionated**: Có cấu trúc và quy tắc rõ ràng
- **Decorator-based**: Sử dụng decorator như Angular
- **TypeScript**: Được xây dựng với TypeScript

### Cấu trúc project NestJS
```
src/
├── modules/         # Feature modules
│   ├── user/
│   │   ├── dto/     # Data Transfer Objects
│   │   ├── entities/# Database entities
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   └── user.module.ts
├── common/          # Shared resources
│   ├── guards/      # Authentication guards
│   ├── interceptors/# Response interceptors
│   ├── pipes/       # Validation pipes
│   └── filters/     # Exception filters
├── config/          # Configuration
├── app.module.ts    # Root module
└── main.ts          # Bootstrap file
```

### NestJS Decorators
```ts
// Controller
@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
  
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}

// Service
@Injectable()
export class UsersService {
  findOne(id: string): User {
    // Logic here
  }
}

// Module
@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

## So sánh Express vs NestJS

| Tiêu chí | Express | NestJS |
|----------|---------|---------|
| **Learning Curve** | Dễ học, đơn giản | Phức tạp hơn, cần hiểu TypeScript |
| **Cấu trúc** | Tự do, không ép buộc | Có cấu trúc rõ ràng, modular |
| **TypeScript** | Hỗ trợ nhưng không bắt buộc | Built-in TypeScript |
| **Performance** | Nhanh, lightweight | Overhead nhỏ, vẫn nhanh |
| **Scalability** | Cần tự thiết kế | Thiết kế sẵn cho large app |
| **Dependency Injection** | Không có | Built-in DI container |
| **Testing** | Cần setup thủ công | Built-in testing utilities |

## Authentication & Authorization

### Express Authentication
```js
// JWT Middleware
const jwt = require('jsonwebtoken');

const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.sendStatus(401);
  }
  
  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};

// Usage
app.get('/protected', authenticateToken, (req, res) => {
  res.json({ user: req.user });
});
```

### NestJS Authentication
```ts
// JWT Strategy
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }
  
  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}

// Guard
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

// Usage
@UseGuards(JwtAuthGuard)
@Get('profile')
getProfile(@Request() req) {
  return req.user;
}
```

## Validation

### Express Validation
```js
const { body, validationResult } = require('express-validator');

const validateUser = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 6 }),
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  }
];

app.post('/register', validateUser, (req, res) => {
  // Logic here
});
```

### NestJS Validation
```ts
// DTO
export class CreateUserDto {
  @IsEmail()
  @IsNotEmpty()
  email: string;
  
  @IsString()
  @MinLength(6)
  password: string;
}

// Controller
@Post()
create(@Body() createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}

// Global validation pipe in main.ts
app.useGlobalPipes(new ValidationPipe());
```

## Error Handling

### Express Error Handling
```js
// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Global error handler
const globalErrorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';
  
  res.status(err.statusCode).json({
    status: err.status,
    message: err.message
  });
};

app.use(globalErrorHandler);
```

### NestJS Error Handling
```ts
// Exception Filter
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    });
  }
}

// Usage
@UseFilters(HttpExceptionFilter)
export class UsersController {}
```

## Testing

### Express Testing
```js
const request = require('supertest');
const app = require('../app');

describe('GET /users', () => {
  it('should return users list', async () => {
    const res = await request(app)
      .get('/users')
      .expect('Content-Type', /json/)
      .expect(200);
      
    expect(res.body).toHaveProperty('users');
  });
});
```

### NestJS Testing
```ts
describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: {
            findAll: jest.fn(() => []),
          },
        },
      ],
    }).compile();
    
    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });
  
  it('should return users', () => {
    expect(controller.findAll()).toEqual([]);
  });
});
```

## Câu hỏi phỏng vấn thường gặp

### Express
1. **Middleware là gì? Giải thích thứ tự thực thi**
2. **Sự khác biệt giữa app.use() và app.get()**
3. **Làm thế nào để handle error trong Express?**
4. **Route parameters vs Query parameters**
5. **Cách implement authentication trong Express**
6. **Express Router hoạt động như thế nào?**
7. **CORS trong Express và cách config**

### NestJS
1. **Dependency Injection trong NestJS hoạt động như thế nào?**
2. **Sự khác biệt giữa @Injectable() và @Controller()**
3. **Module system trong NestJS**
4. **Guards vs Interceptors vs Middlewares**
5. **Pipe validation hoạt động như thế nào?**
6. **Custom decorators trong NestJS**
7. **Exception filters và cách sử dụng**

### So sánh
1. **Khi nào nên chọn Express vs NestJS?**
2. **Performance comparison**
3. **Scalability của từng framework**
4. **Learning curve và ecosystem**
