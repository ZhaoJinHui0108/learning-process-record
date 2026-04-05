# TypeScript 学习笔记

## 目录
1. [TypeScript 基础语法](#typescript-基础语法)
2. [类型系统](#类型系统)
3. [面向对象特性](#面向对象特性)
4. [项目结构](#项目结构)
5. [特殊文件说明](#特殊文件说明)

---

## TypeScript 基础语法

### 基础类型
```typescript
// 布尔值
let isDone: boolean = false;

// 数字
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;

// 字符串
let color: string = "blue";
let template: string = `Hello, ${name}`;

// 数组
let list: number[] = [1, 2, 3];
let genericList: Array<number> = [1, 2, 3];

// 元组
let tuple: [string, number] = ["hello", 42];

// 枚举
enum Color { Red, Green, Blue }
enum Status { Active = 1, Inactive, Pending }

// Any (任意类型)
let notSure: any = 4;
notSure = "maybe a string";

// Void (无返回)
function warnUser(): void {
    console.log("Warning!");
}

// Null 和 Undefined
let u: undefined = undefined;
let n: null = null;

// Never (永不返回)
function error(message: string): never {
    throw new Error(message);
}

// Object
let obj: object = { key: "value" };
```

### 接口
```typescript
interface Person {
    name: string;
    age: number;
    email?: string;  // 可选属性
    readonly id: number;  // 只读属性
}

interface Config {
    width?: number;
    height?: number;
    [propName: string]: any;  // 索引签名
}

function greet(person: Person): string {
    return `Hello, ${person.name}`;
}
```

### 类型别名
```typescript
type Point = { x: number; y: number };
type ID = string | number;
type Callback = (data: string) => void;
```

### 联合类型和交叉类型
```typescript
// 联合类型
let id: string | number;
id = "abc";
id = 123;

// 交叉类型
interface Loggable {
    log(): void;
}
interface Serializable {
    serialize(): string;
}
type LogAndSerializable = Loggable & Serializable;
```

---

## 类型系统

### 函数类型
```typescript
// 函数声明
function add(x: number, y: number): number {
    return x + y;
}

// 函数表达式
let myAdd: (x: number, y: number) => number = function(x, y) {
    return x + y;
};

// 可选参数
function buildName(firstName: string, lastName?: string): string {
    return lastName ? `${firstName} ${lastName}` : firstName;
}

// 默认参数
function buildName2(firstName: string, lastName = "Smith"): string {
    return `${firstName} ${lastName}`;
}

// 剩余参数
function collectNames(...names: string[]): string {
    return names.join(", ");
}

// 函数重载
function overload(x: number): number;
function overload(x: string): string;
function overload(x: any): any {
    return x;
}
```

### 泛型
```typescript
// 泛型函数
function identity<T>(arg: T): T {
    return arg;
}

// 泛型接口
interface GenericIdentity<T> {
    (arg: T): T;
    defaultValue: T;
}

// 泛型约束
interface Lengthwise {
    length: number;
}
function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

// 多类型参数
function pair<K, V>(key: K, value: V): [K, V] {
    return [key, value];
}

// 泛型类
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let stringNumber = new GenericNumber<string>();
stringNumber.zeroValue = "";
stringNumber.add = (x, y) => x + y;
```

### 高级类型

#### 条件类型
```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type ExtractPromise<T> = T extends Promise<infer U> ? U : T;

type Flatten<T> = T extends Array<infer U> ? U : T;
```

#### 映射类型
```typescript
type Readonly<T> = { readonly [P in keyof T]: T[P] };
type Partial<T> = { [P in keyof T]?: T[P] };
type Required<T> = { [P in keyof T]-?: T[P] };
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Record<K extends string, T> = { [P in K]: T };
```

---

## 面向对象特性

### 类
```typescript
class Animal {
    private name: string;
    protected age: number;
    public static species: string = "Unknown";
    
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
    
    move(distance: number = 0): void {
        console.log(`${this.name} moved ${distance}m`);
    }
    
    getName(): string {
        return this.name;
    }
}

class Dog extends Animal {
    private breed: string;
    
    constructor(name: string, age: number, breed: string) {
        super(name, age);
        this.breed = breed;
    }
    
    move(distance: number = 5): void {
        console.log("Running...");
        super.move(distance);
    }
    
    bark(): void {
        console.log("Woof! Woof!");
    }
}

let dog = new Dog("Rex", 3, "German Shepherd");
dog.move(10);
dog.bark();
```

### 抽象类
```typescript
abstract class Shape {
    abstract getArea(): number;
    
    display(): void {
        console.log(`Area: ${this.getArea()}`);
    }
}

class Circle extends Shape {
    constructor(public radius: number) {
        super();
    }
    
    getArea(): number {
        return Math.PI * this.radius ** 2;
    }
}

class Rectangle extends Shape {
    constructor(public width: number, public height: number) {
        super();
    }
    
    getArea(): number {
        return this.width * this.height;
    }
}
```

### 接口
```typescript
interface Printable {
    print(): void;
}

interface Loggable {
    log(): void;
}

class Report implements Printable, Loggable {
    constructor(public content: string) {}
    
    print(): void {
        console.log(this.content);
    }
    
    log(): void {
        console.log(`[LOG] ${this.content}`);
    }
}
```

### 装饰器
```typescript
// 类装饰器
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

// 方法装饰器
function logging(value: boolean) {
    return function (
        target: any,
        propertyKey: string,
        descriptor: PropertyDescriptor
    ) {
        descriptor.value = function (...args: any[]) {
            if (value) {
                console.log(`Calling ${propertyKey} with`, args);
            }
            return descriptor.value.apply(this, args);
        };
    };
}

class Calculator {
    @logging(true)
    add(x: number, y: number): number {
        return x + y;
    }
}
```

---

## 项目结构

### 标准 Node.js/TypeScript 项目结构
```
my-project/
├── src/                      # 源代码目录
│   ├── index.ts             # 项目入口文件
│   ├── app.ts               # 主应用文件
│   ├── config/              # 配置文件
│   │   └── index.ts
│   ├── controllers/         # 控制器 (MVC)
│   │   └── userController.ts
│   ├── models/              # 数据模型
│   │   └── userModel.ts
│   ├── services/            # 业务逻辑
│   │   └── userService.ts
│   ├── routes/              # 路由定义
│   │   └── index.ts
│   ├── middlewares/         # 中间件
│   │   └── auth.ts
│   ├── utils/               # 工具函数
│   │   └── helper.ts
│   ├── types/               # 类型定义
│   │   └── index.d.ts
│   └── constants/          # 常量定义
│       └── index.ts
├── tests/                   # 测试目录
│   ├── unit/               # 单元测试
│   └── integration/        # 集成测试
├── dist/                    # 编译输出目录
├── node_modules/            # 依赖包
├── package.json            # 项目配置
├── tsconfig.json           # TypeScript 配置
├── jest.config.js          # 测试框架配置
├── .eslintrc.js            # ESLint 配置
├── .prettierrc             # Prettier 配置
└── README.md               # 项目文档
```

### React + TypeScript 项目结构
```
my-react-app/
├── public/                  # 静态资源
│   ├── index.html
│   └── favicon.ico
├── src/                     # 源代码目录
│   ├── index.tsx           # 入口文件
│   ├── App.tsx             # 主应用组件
│   ├── components/         # 公共组件
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   └── Button.css
│   │   └── Modal/
│   ├── pages/               # 页面组件
│   │   ├── Home/
│   │   └── About/
│   ├── hooks/               # 自定义 Hooks
│   │   └── useAuth.ts
│   ├── contexts/            # React Context
│   │   └── AuthContext.tsx
│   ├── services/           # API 服务
│   │   └── api.ts
│   ├── stores/              # 状态管理
│   │   └── useStore.ts
│   ├── types/               # 类型定义
│   │   └── index.ts
│   ├── utils/               # 工具函数
│   └── styles/              # 全局样式
├── package.json
├── tsconfig.json
├── vite.config.ts          # Vite 配置
└── README.md
```

### Angular 项目结构
```
my-angular-app/
├── src/
│   ├── app/
│   │   ├── core/                    # 核心模块 (单例服务)
│   │   │   ├── services/
│   │   │   ├── guards/
│   │   │   └── interceptors/
│   │   ├── shared/                  # 共享模块
│   │   │   ├── components/
│   │   │   ├── directives/
│   │   │   ├── pipes/
│   │   │   └── shared.module.ts
│   │   ├── features/                # 功能模块
│   │   │   ├── dashboard/
│   │   │   └── settings/
│   │   ├── app.component.ts         # 根组件
│   │   ├── app.module.ts           # 根模块
│   │   └── app-routing.module.ts   # 路由模块
│   ├── assets/
│   ├── environments/
│   ├── styles.scss
│   └── index.html
├── angular.json
├── package.json
└── tsconfig.json
```

---

## 特殊文件说明

### package.json
Node.js项目的核心配置文件，定义项目依赖和脚本。
```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "Project description",
  "main": "dist/index.js",
  "scripts": {
    "dev": "ts-node src/index.ts",
    "build": "tsc",
    "test": "jest",
    "lint": "eslint src/**/*.ts"
  },
  "dependencies": {
    "express": "^4.18.0",
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "typescript": "^5.0.0",
    "ts-node": "^10.9.0"
  }
}
```

### tsconfig.json
TypeScript编译器的配置文件。
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
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### .eslintrc.js
ESLint代码检查配置文件。
```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module',
    project: './tsconfig.json'
  },
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended'
  ],
  rules: {
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
    'no-console': 'warn'
  },
  env: {
    node: true,
    es6: true
  }
};
```

### jest.config.js
Jest测试框架配置文件。
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  transform: {
    '^.+\\.ts$': 'ts-jest'
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/__tests__/**'
  ],
  coverageDirectory: 'coverage',
  moduleFileExtensions: ['ts', 'js', 'json']
};
```

### vite.config.ts
Vite构建工具配置文件。
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  },
  resolve: {
    alias: {
      '@': '/src'
    }
  }
});
```

### .gitignore
Git忽略文件配置。
```
# 依赖
node_modules/
package-lock.json

# 构建输出
dist/
build/

# IDE
.vscode/
.idea/

# 日志
*.log
npm-debug.log*

# 操作系统
.DS_Store
Thumbs.db

# 测试覆盖率
coverage/

# 环境变量
.env
.env.local
```

### tsconfig.node.json
Node.js环境的TypeScript配置（用于Vite等工具）。
```json
{
  "compilerOptions": {
    "composite": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

### tsconfig.app.json
应用代码的TypeScript配置。
```json
{
  "extends": "@vue/tsconfig/tsconfig.dom.json",
  "compilerOptions": {
    "composite": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx"],
  "exclude": ["src/**/__tests__/**"]
}
```

### jsconfig.json / tsconfig.json (monorepo)
Monorepo项目配置。
```json
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/web" },
    { "path": "./packages/cli" }
  ]
}
```

### vite-env.d.ts
Vite环境类型声明文件。
```typescript
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string;
  readonly VITE_API_BASE_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```
