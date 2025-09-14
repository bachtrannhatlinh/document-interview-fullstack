# TYPESCRIPT ADVANCED - INTERVIEW GUIDE

## 🎯 TYPESCRIPT CORE CONCEPTS

### 1. Type System Fundamentals

#### **Basic Types**
```typescript
// Primitive types
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let data: null = null;
let value: undefined = undefined;

// Array types
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"];

// Tuple types
let person: [string, number] = ["John", 30];
let coordinates: [number, number, number] = [10, 20, 30];

// Enum types
enum Color {
  Red = "red",
  Green = "green", 
  Blue = "blue"
}
let favoriteColor: Color = Color.Red;

// Any, Unknown, Never
let anything: any = 42; // Avoid using any
let userInput: unknown = getData(); // Safer than any
let neverReturns: never = throwError("Error"); // Never returns
```

#### **Union & Intersection Types**
```typescript
// Union types
type Status = "pending" | "approved" | "rejected";
type ID = string | number;

function processId(id: ID) {
  if (typeof id === "string") {
    return id.toUpperCase(); // Type narrowing
  }
  return id.toFixed(2);
}

// Intersection types
interface Printable {
  print(): void;
}

interface Saveable {
  save(): void;
}

type Document = Printable & Saveable;

class PDFDocument implements Document {
  print() { console.log("Printing PDF"); }
  save() { console.log("Saving PDF"); }
}
```

---

## 🏗️ ADVANCED TYPE FEATURES

### 2. Interfaces vs Type Aliases

#### **Interfaces**
```typescript
interface User {
  readonly id: number; // Cannot be modified
  name: string;
  email?: string; // Optional property
  [key: string]: any; // Index signature
}

// Interface merging (declaration merging)
interface User {
  createdAt: Date;
}

// Now User has: id, name, email?, createdAt, [key: string]: any

// Interface inheritance
interface Admin extends User {
  permissions: string[];
  role: "admin" | "superadmin";
}

// Function interface
interface SearchFunction {
  (source: string, substring: string): boolean;
}

let mySearch: SearchFunction = function(src, sub) {
  return src.search(sub) > -1;
};
```

#### **Type Aliases**
```typescript
type User = {
  id: number;
  name: string;
  email?: string;
};

// Type aliases can use unions
type Status = "loading" | "success" | "error";

// Computed property names
type EventHandlers = {
  [K in keyof WindowEventMap as `on${Capitalize<K>}`]?: (
    event: WindowEventMap[K]
  ) => void;
};

// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;

// Template literal types
type EventName<T extends string> = `on${Capitalize<T>}`;
type ClickEvent = EventName<"click">; // "onClick"
```

### 3. Generic Types

#### **Basic Generics**
```typescript
// Generic functions
function identity<T>(arg: T): T {
  return arg;
}

let stringResult = identity<string>("hello");
let numberResult = identity(42); // Type inference

// Generic interfaces
interface Repository<T> {
  find(id: string): T | null;
  save(entity: T): void;
  delete(id: string): boolean;
}

class UserRepository implements Repository<User> {
  find(id: string): User | null {
    // Implementation
    return null;
  }
  
  save(user: User): void {
    // Implementation
  }
  
  delete(id: string): boolean {
    // Implementation
    return true;
  }
}

// Generic constraints
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("hello"); // ✅ String has length
logLength([1, 2, 3]); // ✅ Array has length
// logLength(42); // ❌ Number doesn't have length
```

#### **Advanced Generic Patterns**
```typescript
// Generic utility types
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

// Usage examples
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PartialUser = Partial<User>; // All properties optional
type UserWithoutPassword = Omit<User, "password">;
type UserCredentials = Pick<User, "email" | "password">;

// Generic class with constraints
class ApiClient<T extends { id: string | number }> {
  constructor(private baseUrl: string) {}
  
  async getById(id: T["id"]): Promise<T> {
    const response = await fetch(`${this.baseUrl}/${id}`);
    return response.json();
  }
  
  async create(entity: Omit<T, "id">): Promise<T> {
    const response = await fetch(this.baseUrl, {
      method: 'POST',
      body: JSON.stringify(entity)
    });
    return response.json();
  }
}
```

---

## 🔧 UTILITY TYPES & MAPPED TYPES

### 4. Built-in Utility Types

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  isActive: boolean;
}

// Partial - makes all properties optional
type UserUpdate = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; isActive?: boolean; }

// Required - makes all properties required
type RequiredUser = Required<Partial<User>>;

// Pick - select specific properties
type UserSummary = Pick<User, "id" | "name" | "email">;
// { id: number; name: string; email: string; }

// Omit - exclude specific properties
type UserWithoutId = Omit<User, "id">;
// { name: string; email: string; age: number; isActive: boolean; }

// Record - create object type with specific keys and values
type UserRoles = Record<"admin" | "user" | "guest", string[]>;
// { admin: string[]; user: string[]; guest: string[]; }

// Exclude - exclude types from union
type Status = "pending" | "approved" | "rejected";
type ActiveStatus = Exclude<Status, "rejected">; // "pending" | "approved"

// Extract - extract types from union
type StringOrNumber = string | number | boolean;
type StringType = Extract<StringOrNumber, string>; // string

// NonNullable - exclude null and undefined
type NonNullableString = NonNullable<string | null | undefined>; // string

// ReturnType - get return type of function
function getUser(): User { return {} as User; }
type UserReturnType = ReturnType<typeof getUser>; // User

// Parameters - get parameters of function
function updateUser(id: number, data: Partial<User>): void {}
type UpdateUserParams = Parameters<typeof updateUser>; // [number, Partial<User>]
```

### 5. Custom Mapped Types

```typescript
// Create read-only version of all properties
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Create mutable version (remove readonly)
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

// Add prefix to all property names
type PrefixKeys<T, Prefix extends string> = {
  [K in keyof T as `${Prefix}${string & K}`]: T[K];
};

type PrefixedUser = PrefixKeys<User, "user_">;
// { user_id: number; user_name: string; user_email: string; }

// Convert all properties to strings
type Stringify<T> = {
  [K in keyof T]: string;
};

// Conditional mapped type
type NullableKeys<T> = {
  [K in keyof T]: T[K] extends null | undefined ? K : never;
}[keyof T];

type OptionalKeys<T> = {
  [K in keyof T]: {} extends Pick<T, K> ? K : never;
}[keyof T];

// Deep partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```

---

## 🎯 INTERVIEW QUESTIONS & ANSWERS

### 1. **"Interface vs Type Alias - Khi nào dùng gì?"**

```typescript
// Use INTERFACE when:
// 1. Defining object shapes
interface User {
  id: number;
  name: string;
}

// 2. Declaration merging (extending from libraries)
interface Window {
  customProperty: string;
}

// 3. Class implementation
class UserService implements User {
  id: number;
  name: string;
}

// Use TYPE when:
// 1. Union types
type Status = "loading" | "success" | "error";

// 2. Computed/conditional types
type Keys<T> = keyof T;

// 3. Primitive aliases
type ID = string | number;
```

### 2. **"Generic constraints vs Conditional types?"**

```typescript
// Generic constraints - limit what T can be
function process<T extends string | number>(value: T): T {
  return value;
}

// Conditional types - different behavior based on type
type ApiResponse<T> = T extends string 
  ? { message: T } 
  : { data: T };

type StringResponse = ApiResponse<string>; // { message: string }
type NumberResponse = ApiResponse<number>; // { data: number }
```

### 3. **"any vs unknown vs never?"**

```typescript
// any - disables type checking (avoid!)
let anything: any = 42;
anything.foo.bar; // No error, but dangerous

// unknown - safe alternative to any
let userInput: unknown = getValue();
if (typeof userInput === "string") {
  console.log(userInput.toUpperCase()); // ✅ Type narrowing
}

// never - represents values that never occur
function throwError(message: string): never {
  throw new Error(message);
}

type Unreachable = string & number; // never (impossible type)
```

### 4. **"Type assertion vs Type guards?"**

```typescript
// Type assertion - telling TypeScript what type something is
let someValue: unknown = "hello";
let strLength: number = (someValue as string).length;

// Type guards - runtime checks that narrow types
function isString(value: unknown): value is string {
  return typeof value === "string";
}

if (isString(someValue)) {
  console.log(someValue.toUpperCase()); // TypeScript knows it's string
}

// Discriminated unions with type guards
interface Circle {
  kind: "circle";
  radius: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

type Shape = Circle | Rectangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // TypeScript knows it's Circle
    case "rectangle":
      return shape.width * shape.height; // TypeScript knows it's Rectangle
    default:
      const exhaustive: never = shape; // Ensures all cases handled
      return exhaustive;
  }
}
```

---

## 💻 REAL-WORLD EXAMPLES

### Express.js với TypeScript

```typescript
// types/express.d.ts - Extend Express types
import { User } from '../models/User';

declare global {
  namespace Express {
    interface Request {
      user?: User;
    }
  }
}

// middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

interface JWTPayload {
  userId: string;
  email: string;
}

export const authMiddleware = async (
  req: Request, 
  res: Response, 
  next: NextFunction
): Promise<void> => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      res.status(401).json({ error: 'No token provided' });
      return;
    }

    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
    const user = await getUserById(payload.userId);
    
    if (!user) {
      res.status(401).json({ error: 'Invalid token' });
      return;
    }

    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

// controllers/userController.ts
import { Request, Response } from 'express';
import { body, validationResult } from 'express-validator';

// DTO (Data Transfer Object)
interface CreateUserDTO {
  name: string;
  email: string;
  password: string;
}

interface UpdateUserDTO {
  name?: string;
  email?: string;
}

export const createUser = [
  // Validation middleware
  body('name').isLength({ min: 2 }).trim(),
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 6 }),
  
  async (req: Request<{}, {}, CreateUserDTO>, res: Response): Promise<void> => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      res.status(400).json({ errors: errors.array() });
      return;
    }

    try {
      const { name, email, password } = req.body;
      const user = await createUserService({ name, email, password });
      
      res.status(201).json({ 
        user: {
          id: user.id,
          name: user.name,
          email: user.email
        }
      });
    } catch (error) {
      res.status(500).json({ error: 'Failed to create user' });
    }
  }
];

export const updateUser = async (
  req: Request<{ id: string }, {}, UpdateUserDTO>, 
  res: Response
): Promise<void> => {
  try {
    const { id } = req.params;
    const updates = req.body;
    
    const user = await updateUserService(id, updates);
    
    if (!user) {
      res.status(404).json({ error: 'User not found' });
      return;
    }

    res.json({ user });
  } catch (error) {
    res.status(500).json({ error: 'Failed to update user' });
  }
};
```

### React với TypeScript

```typescript
// types/api.ts
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

export interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

export interface PaginatedResponse<T> extends ApiResponse<T[]> {
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// hooks/useApi.ts
import { useState, useEffect } from 'react';

interface UseApiState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

interface UseApiReturn<T> extends UseApiState<T> {
  refetch: () => Promise<void>;
}

export function useApi<T>(url: string): UseApiReturn<T> {
  const [state, setState] = useState<UseApiState<T>>({
    data: null,
    loading: true,
    error: null
  });

  const fetchData = async (): Promise<void> => {
    try {
      setState(prev => ({ ...prev, loading: true, error: null }));
      
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const data: T = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ 
        data: null, 
        loading: false, 
        error: error instanceof Error ? error.message : 'Unknown error' 
      });
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return { ...state, refetch: fetchData };
}

// components/UserList.tsx
import React, { useMemo } from 'react';
import { useApi } from '../hooks/useApi';
import { PaginatedResponse, User } from '../types/api';

interface UserListProps {
  searchTerm?: string;
  onUserSelect?: (user: User) => void;
}

const UserList: React.FC<UserListProps> = ({ 
  searchTerm = '', 
  onUserSelect 
}) => {
  const { data, loading, error, refetch } = useApi<PaginatedResponse<User>>(
    `/api/users?search=${encodeURIComponent(searchTerm)}`
  );

  const filteredUsers = useMemo(() => {
    if (!data?.data) return [];
    return data.data.filter(user => 
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [data?.data, searchTerm]);

  if (loading) return <div>Loading users...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!data?.data?.length) return <div>No users found</div>;

  return (
    <div className="user-list">
      <div className="user-list__header">
        <h2>Users ({data.pagination.total})</h2>
        <button onClick={refetch}>Refresh</button>
      </div>
      
      <div className="user-list__items">
        {filteredUsers.map(user => (
          <div 
            key={user.id} 
            className="user-item"
            onClick={() => onUserSelect?.(user)}
          >
            {user.avatar && (
              <img src={user.avatar} alt={user.name} className="user-avatar" />
            )}
            <div className="user-info">
              <h3>{user.name}</h3>
              <p>{user.email}</p>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};

export default UserList;
```

### Database Models với TypeScript

```typescript
// models/User.ts
import { Schema, model, Document } from 'mongoose';

// Base interface for User data
export interface IUser {
  name: string;
  email: string;
  password: string;
  role: 'user' | 'admin';
  createdAt: Date;
  updatedAt: Date;
}

// Mongoose document interface
export interface IUserDocument extends IUser, Document {
  comparePassword(password: string): Promise<boolean>;
  generateToken(): string;
}

// Mongoose model interface
export interface IUserModel extends Model<IUserDocument> {
  findByEmail(email: string): Promise<IUserDocument | null>;
  findActiveUsers(): Promise<IUserDocument[]>;
}

const userSchema = new Schema<IUserDocument>({
  name: { type: String, required: true, trim: true },
  email: { 
    type: String, 
    required: true, 
    unique: true, 
    lowercase: true 
  },
  password: { type: String, required: true },
  role: { 
    type: String, 
    enum: ['user', 'admin'], 
    default: 'user' 
  }
}, {
  timestamps: true
});

// Instance methods
userSchema.methods.comparePassword = async function(
  this: IUserDocument, 
  password: string
): Promise<boolean> {
  return bcrypt.compare(password, this.password);
};

userSchema.methods.generateToken = function(this: IUserDocument): string {
  return jwt.sign(
    { userId: this._id, email: this.email },
    process.env.JWT_SECRET!,
    { expiresIn: '7d' }
  );
};

// Static methods
userSchema.statics.findByEmail = function(
  this: IUserModel,
  email: string
): Promise<IUserDocument | null> {
  return this.findOne({ email });
};

userSchema.statics.findActiveUsers = function(
  this: IUserModel
): Promise<IUserDocument[]> {
  return this.find({ isActive: true });
};

export const User = model<IUserDocument, IUserModel>('User', userSchema);
```

---

## ⚡ PERFORMANCE & BEST PRACTICES

### 1. Compiler Configuration

```json
// tsconfig.json
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
    
    // Performance optimizations
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",
    
    // Strict type checking
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    
    // Module resolution
    "moduleResolution": "node",
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "@/types/*": ["src/types/*"],
      "@/utils/*": ["src/utils/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 2. Type-only Imports

```typescript
// Performance optimization - import only types
import type { User } from './types/User';
import type { ApiResponse } from './types/api';

// Regular import for runtime values
import { validateUser } from './utils/validation';

// Mixed imports
import { type User, createUser } from './services/userService';
```

### 3. Declaration Files

```typescript
// types/global.d.ts
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      NODE_ENV: 'development' | 'production' | 'test';
      JWT_SECRET: string;
      DATABASE_URL: string;
      PORT: string;
    }
  }
}

// Module augmentation
declare module 'express-serve-static-core' {
  interface Request {
    user?: User;
  }
}

// Ambient module declarations
declare module '*.json' {
  const value: any;
  export default value;
}

declare module 'some-untyped-library' {
  export function doSomething(param: string): boolean;
}
```

---

## 🚀 TESTING với TypeScript

```typescript
// __tests__/userService.test.ts
import { describe, it, expect, beforeEach, jest } from '@jest/globals';
import { createUser, getUserById } from '../src/services/userService';
import { User } from '../src/models/User';

// Mock dependencies
jest.mock('../src/models/User');
const mockedUser = jest.mocked(User);

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('createUser', () => {
    it('should create user with valid data', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123'
      };

      const mockUser = {
        id: '1',
        ...userData,
        save: jest.fn().mockResolvedValue(userData)
      };

      mockedUser.prototype.constructor = jest.fn().mockReturnValue(mockUser);
      
      const result = await createUser(userData);
      
      expect(result).toEqual(expect.objectContaining({
        name: userData.name,
        email: userData.email
      }));
      expect(mockUser.save).toHaveBeenCalled();
    });

    it('should throw error for duplicate email', async () => {
      const userData = {
        name: 'John Doe', 
        email: 'john@example.com',
        password: 'password123'
      };

      mockedUser.prototype.save = jest.fn().mockRejectedValue({
        code: 11000, // MongoDB duplicate key error
        keyValue: { email: userData.email }
      });

      await expect(createUser(userData)).rejects.toThrow('Email already exists');
    });
  });
});

// Type-safe test utilities
interface MockUser extends Partial<User> {
  id: string;
  name: string;
  email: string;
}

function createMockUser(overrides: Partial<MockUser> = {}): MockUser {
  return {
    id: '1',
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  };
}

// Usage in tests
const mockUser = createMockUser({ 
  name: 'Custom Name',
  email: 'custom@example.com' 
});
```

---

## ✅ CHECKLIST CHUẨN BỊ PHỎNG VẤN

### **Basic Concepts:**
- [ ] Interface vs Type aliases
- [ ] Union vs Intersection types
- [ ] Generic constraints & conditional types
- [ ] Utility types (Partial, Pick, Omit, etc.)
- [ ] Type guards & assertions

### **Advanced Topics:**
- [ ] Mapped types & template literals
- [ ] Declaration merging & module augmentation
- [ ] Discriminated unions
- [ ] Recursive types
- [ ] Brand types (nominal typing)

### **Practical Skills:**
- [ ] Express.js với TypeScript
- [ ] React props & hooks typing
- [ ] Database models typing
- [ ] API client typing
- [ ] Test setup với TypeScript

### **Configuration:**
- [ ] tsconfig.json optimization
- [ ] Path mapping & aliases
- [ ] Declaration files (.d.ts)
- [ ] Build process integration
- [ ] ESLint + TypeScript rules

---

## 🎯 COMMON INTERVIEW MISTAKES

### ❌ **Bad Practices:**
```typescript
// 1. Using 'any' everywhere
function processData(data: any): any {
  return data.something;
}

// 2. Not using strict mode
let user; // implicitly 'any'
user.name = "John"; // No type checking

// 3. Over-complicated types
type ComplexType<T> = T extends infer U 
  ? U extends string 
    ? `prefix_${U}` 
    : never 
  : never;
```

### ✅ **Good Practices:**
```typescript
// 1. Use proper types
function processData<T>(data: T): T {
  return data;
}

// 2. Enable strict mode, use unknown
let user: unknown;
if (typeof user === 'object' && user !== null) {
  // Safe to use user
}

// 3. Keep types simple and readable
type PrefixedString<T extends string> = `prefix_${T}`;
```

---

Với file này, bạn sẽ có đầy đủ kiến thức TypeScript từ cơ bản đến nâng cao cho phỏng vấn full-stack developer!
