# 14. TypeScript

## Tổng quan về TypeScript

### Lợi ích chính
- **Type Safety**: Phát hiện lỗi khi compile-time, giảm bug runtime
- **IntelliSense**: Tự động complete, refactor, navigate code
- **Maintainability**: Dễ bảo trì, mở rộng dự án lớn
- **Documentation**: Code tự document thông qua types
- **Refactoring**: An toàn khi refactor code lớn

### Nhược điểm
- **Learning Curve**: Cần thời gian học type system
- **Compile Time**: Thêm bước build process
- **Boilerplate**: Nhiều type definitions
- **Library Support**: Không phải library nào cũng có types

## Setup & Configuration

### Cài đặt cơ bản
```bash
# Install TypeScript globally
npm install -g typescript

# Install for project
npm install typescript @types/node --save-dev
npm install ts-node nodemon --save-dev

# Init tsconfig.json
npx tsc --init
```

### tsconfig.json quan trọng
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Package.json scripts
```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts",
    "watch": "nodemon --exec ts-node src/index.ts",
    "type-check": "tsc --noEmit"
  }
}
```

## Basic Types & Syntax

### Primitive Types
```ts
// Basic types
let str: string = "hello";
let num: number = 42;
let bool: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b", "c"];

// Tuple
let tuple: [string, number] = ["hello", 42];

// Any (avoid when possible)
let anything: any = "could be anything";

// Unknown (safer than any)
let userInput: unknown;
if (typeof userInput === "string") {
  console.log(userInput.toUpperCase());
}

// Never (for functions that never return)
function throwError(): never {
  throw new Error("Something went wrong");
}
```

### Union & Intersection Types
```ts
// Union types
type StringOrNumber = string | number;
let value: StringOrNumber = "hello";
value = 42; // OK

// Intersection types
type Person = { name: string };
type Employee = { employeeId: number };
type PersonEmployee = Person & Employee;

const worker: PersonEmployee = {
  name: "John",
  employeeId: 123
};
```

### Literal Types
```ts
// String literals
type Status = "pending" | "approved" | "rejected";
let orderStatus: Status = "pending";

// Numeric literals
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

// Boolean literals
type SuccessResponse = { success: true; data: any };
type ErrorResponse = { success: false; error: string };
type Response = SuccessResponse | ErrorResponse;
```

## Interfaces vs Types

### Interface
```ts
interface User {
  readonly id: number;
  name: string;
  email?: string; // optional
  age: number;
}

// Interface inheritance
interface Admin extends User {
  permissions: string[];
}

// Interface merging (declaration merging)
interface User {
  lastLogin?: Date;
}

// Method signatures
interface UserService {
  getUser(id: number): Promise<User>;
  createUser(user: Omit<User, 'id'>): Promise<User>;
  updateUser(id: number, updates: Partial<User>): Promise<User>;
}
```

### Type Aliases
```ts
type User = {
  readonly id: number;
  name: string;
  email?: string;
  age: number;
};

// Type unions
type Theme = "light" | "dark";
type Size = "small" | "medium" | "large";

// Computed types
type UserKeys = keyof User; // "id" | "name" | "email" | "age"
type UserName = User['name']; // string

// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;
```

### Khi nào dùng Interface vs Type?
```ts
// Use Interface for:
// 1. Object shapes that might be extended
// 2. API contracts
// 3. Class implementations

interface API {
  endpoint: string;
}

class UserAPI implements API {
  endpoint = "/users";
}

// Use Type for:
// 1. Unions, intersections
// 2. Computed types
// 3. Type transformations

type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type APIConfig<T> = {
  method: HttpMethod;
  data?: T;
};
```

## Advanced Types

### Generics
```ts
// Generic functions
function identity<T>(arg: T): T {
  return arg;
}

// Generic interfaces
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  create(item: Omit<T, 'id'>): Promise<T>;
  update(id: string, updates: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// Generic constraints
interface Identifiable {
  id: string;
}

function updateEntity<T extends Identifiable>(
  entity: T,
  updates: Partial<T>
): T {
  return { ...entity, ...updates };
}

// Multiple generics
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}
```

### Utility Types
```ts
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;

// Required - makes all properties required
type RequiredUser = Required<User>;

// Pick - select specific properties
type UserProfile = Pick<User, 'name' | 'email'>;

// Omit - exclude specific properties
type CreateUser = Omit<User, 'id'>;

// Record - create object type with specific keys
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>;

// Exclude - exclude types from union
type NonAdminRoles = Exclude<'admin' | 'user' | 'guest', 'admin'>;

// Extract - extract types from union
type AdminRole = Extract<'admin' | 'user' | 'guest', 'admin'>;

// ReturnType - get return type of function
function getUser(): Promise<User> {
  return Promise.resolve({} as User);
}
type GetUserReturn = ReturnType<typeof getUser>; // Promise<User>
```

### Mapped Types
```ts
// Make all properties optional
type Optional<T> = {
  [K in keyof T]?: T[K];
};

// Make all properties readonly
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

// Transform property types
type Stringify<T> = {
  [K in keyof T]: string;
};

// Conditional mapped types
type NonFunctionKeys<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];
```

## Functions & Classes

### Function Types
```ts
// Function type annotations
function add(a: number, b: number): number {
  return a + b;
}

// Function expressions
const multiply = (a: number, b: number): number => a * b;

// Function types as variables
type MathOperation = (a: number, b: number) => number;
const divide: MathOperation = (a, b) => a / b;

// Optional parameters
function greet(name: string, title?: string): string {
  return title ? `Hello ${title} ${name}` : `Hello ${name}`;
}

// Default parameters
function createUser(name: string, role: string = "user"): User {
  return { id: Date.now(), name, role } as User;
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0);
}

// Overloads
function format(value: string): string;
function format(value: number): string;
function format(value: string | number): string {
  return value.toString();
}
```

### Classes
```ts
class User {
  // Property declarations
  private _id: number;
  protected name: string;
  public email: string;
  
  // Constructor
  constructor(name: string, email: string) {
    this._id = Math.random();
    this.name = name;
    this.email = email;
  }
  
  // Getters/Setters
  get id(): number {
    return this._id;
  }
  
  // Methods
  public greet(): string {
    return `Hello, I'm ${this.name}`;
  }
  
  // Static methods
  static createGuest(): User {
    return new User("Guest", "guest@example.com");
  }
}

// Abstract classes
abstract class Animal {
  abstract makeSound(): string;
  
  move(): string {
    return "Moving...";
  }
}

class Dog extends Animal {
  makeSound(): string {
    return "Woof!";
  }
}

// Implementing interfaces
interface Flyable {
  fly(): void;
}

class Bird implements Animal, Flyable {
  makeSound(): string {
    return "Chirp!";
  }
  
  fly(): void {
    console.log("Flying...");
  }
}
```

## TypeScript với Node.js & Express

### Express với TypeScript
```ts
import express, { Request, Response, NextFunction } from 'express';

const app = express();

// Custom request interface
interface CustomRequest extends Request {
  user?: User;
}

// Route handlers with types
app.get('/users/:id', async (req: Request, res: Response) => {
  const { id } = req.params;
  try {
    const user = await userService.findById(id);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: 'User not found' });
  }
});

// Middleware with types
const authMiddleware = (
  req: CustomRequest, 
  res: Response, 
  next: NextFunction
) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  // Verify token and set user
  req.user = { id: 1, name: 'John' } as User;
  next();
};

// Error handling
app.use((error: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(error.stack);
  res.status(500).json({ error: 'Internal server error' });
});
```

### Database Models (Mongoose)
```ts
import mongoose, { Document, Schema } from 'mongoose';

interface IUser extends Document {
  name: string;
  email: string;
  age: number;
  createdAt: Date;
}

const userSchema = new Schema<IUser>({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, required: true },
  createdAt: { type: Date, default: Date.now }
});

export const User = mongoose.model<IUser>('User', userSchema);
```

### Service Layer với Types
```ts
// DTOs (Data Transfer Objects)
export interface CreateUserDto {
  name: string;
  email: string;
  age: number;
}

export interface UpdateUserDto {
  name?: string;
  email?: string;
  age?: number;
}

// Service class
export class UserService {
  async createUser(userData: CreateUserDto): Promise<User> {
    const user = new User(userData);
    return await user.save();
  }
  
  async findById(id: string): Promise<User | null> {
    return await User.findById(id);
  }
  
  async updateUser(id: string, updates: UpdateUserDto): Promise<User | null> {
    return await User.findByIdAndUpdate(id, updates, { new: true });
  }
  
  async deleteUser(id: string): Promise<boolean> {
    const result = await User.findByIdAndDelete(id);
    return !!result;
  }
}
```

## Type Guards & Narrowing

### Type Guards
```ts
// typeof guards
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

// instanceof guards
class ApiError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
  }
}

function isApiError(error: unknown): error is ApiError {
  return error instanceof ApiError;
}

// Custom type guards
interface Fish {
  swim(): void;
}

interface Bird {
  fly(): void;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

// Using type guards
function handlePet(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim(); // TypeScript knows pet is Fish
  } else {
    pet.fly(); // TypeScript knows pet is Bird
  }
}
```

### Discriminated Unions
```ts
interface LoadingState {
  status: 'loading';
}

interface SuccessState {
  status: 'success';
  data: any;
}

interface ErrorState {
  status: 'error';
  error: string;
}

type AsyncState = LoadingState | SuccessState | ErrorState;

function handleState(state: AsyncState) {
  switch (state.status) {
    case 'loading':
      console.log('Loading...');
      break;
    case 'success':
      console.log('Data:', state.data); // TypeScript knows data exists
      break;
    case 'error':
      console.log('Error:', state.error); // TypeScript knows error exists
      break;
  }
}
```

## Error Handling với TypeScript

### Custom Error Types
```ts
abstract class AppError extends Error {
  abstract statusCode: number;
  
  constructor(message: string) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

class ValidationError extends AppError {
  statusCode = 400;
  
  constructor(public field: string, message: string) {
    super(`Validation failed for ${field}: ${message}`);
  }
}

class NotFoundError extends AppError {
  statusCode = 404;
  
  constructor(resource: string) {
    super(`${resource} not found`);
  }
}

// Result pattern
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

async function fetchUser(id: string): Promise<Result<User, NotFoundError>> {
  try {
    const user = await userService.findById(id);
    if (!user) {
      return { success: false, error: new NotFoundError('User') };
    }
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: error as NotFoundError };
  }
}
```

## Testing với TypeScript

### Jest + TypeScript
```ts
// user.test.ts
import { UserService } from '../services/UserService';
import { User } from '../models/User';

describe('UserService', () => {
  let userService: UserService;
  
  beforeEach(() => {
    userService = new UserService();
  });
  
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        age: 30
      };
      
      const user = await userService.createUser(userData);
      
      expect(user).toHaveProperty('id');
      expect(user.name).toBe(userData.name);
      expect(user.email).toBe(userData.email);
    });
    
    it('should throw error for invalid email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'invalid-email',
        age: 30
      };
      
      await expect(userService.createUser(userData)).rejects.toThrow();
    });
  });
});

// Mock types
const mockUser: jest.Mocked<User> = {
  id: '123',
  name: 'Mock User',
  email: 'mock@example.com',
  age: 25,
  save: jest.fn(),
  toJSON: jest.fn()
} as any;
```

## Best Practices

### 1. Type Organization
```ts
// types/user.types.ts
export interface User {
  id: string;
  name: string;
  email: string;
}

export type CreateUserRequest = Omit<User, 'id'>;
export type UpdateUserRequest = Partial<CreateUserRequest>;
export type UserResponse = User;
```

### 2. Strict Configuration
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noImplicitThis": true
  }
}
```

### 3. Avoid Common Pitfalls
```ts
// ❌ Don't use any
function processData(data: any) {
  return data.someProperty;
}

// ✅ Use unknown and type guards
function processData(data: unknown) {
  if (typeof data === 'object' && data !== null && 'someProperty' in data) {
    return (data as { someProperty: any }).someProperty;
  }
}

// ❌ Don't use function overloads unnecessarily
function format(value: string): string;
function format(value: number): string;

// ✅ Use union types
function format(value: string | number): string {
  return value.toString();
}
```

## Câu hỏi phỏng vấn thường gặp

### 1. TypeScript vs JavaScript?
- **TypeScript**: Static typing, compile-time error checking, better IDE support
- **JavaScript**: Dynamic typing, no compilation step, smaller learning curve

### 2. Interface vs Type aliases?
- **Interface**: Extensible, declaration merging, better for object shapes
- **Type**: Unions, computed types, more flexible for complex types

### 3. Generic là gì và khi nào sử dụng?
- **Generic**: Cho phép tạo reusable components với nhiều types
- **Khi nào**: Functions, classes, interfaces cần hoạt động với nhiều types

### 4. any vs unknown vs never?
- **any**: Tắt type checking (avoid khi có thể)
- **unknown**: Type-safe version của any, cần type narrowing
- **never**: Cho functions không bao giờ return hoặc unreachable code

### 5. Type Guards là gì?
- **Type Guards**: Functions hoặc expressions giúp TypeScript narrow types
- **Ví dụ**: typeof, instanceof, custom type guard functions

### 6. Utility Types nào hay dùng nhất?
- **Partial<T>**: Makes all properties optional
- **Pick<T, K>**: Select specific properties
- **Omit<T, K>**: Exclude specific properties
- **Record<K, T>**: Create object type with specific keys
