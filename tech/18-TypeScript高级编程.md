# TypeScript 高级编程

> 📅 创建时间：2026-05-30
> 🎯 目标：掌握 TypeScript 高级特性

---

## 一、高级类型

### 1.1 条件类型
```typescript
// 基础条件类型
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// 分发条件类型
type ToArray<T> = T extends any ? T[] : never;

type C = ToArray<string | number>;  // string[] | number[]

// infer 关键字
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type D = ReturnType<() => string>;  // string
type E = ReturnType<(x: number) => boolean>;  // boolean

// 递归条件类型
type DeepReadonly<T> = {
    readonly [P in keyof T]: T[P] extends object 
        ? T[P] extends Function 
            ? T[P] 
            : DeepReadonly<T[P]>
        : T[P];
};

interface User {
    name: string;
    address: {
        city: string;
        street: string;
    };
}

type ReadonlyUser = DeepReadonly<User>;
// {
//     readonly name: string;
//     readonly address: {
//         readonly city: string;
//         readonly street: string;
//     };
// }
```

### 1.2 映射类型
```typescript
// 基础映射类型
type Optional<T> = {
    [P in keyof T]?: T[P];
};

type Readonly2<T> = {
    readonly [P in keyof T]: T[P];
};

// 键重映射 (as)
type Getters<T> = {
    [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// {
//     getName: () => string;
//     getAge: () => number;
// }

// 过滤属性
type FilterByType<T, U> = {
    [P in keyof T as T[P] extends U ? P : never]: T[P];
};

type StringProps = FilterByType<Person, string>;
// { name: string }
```

### 1.3 模板字面量类型
```typescript
// 基础模板
type Greeting = `Hello, ${string}!`;

// 组合
type Color = 'red' | 'blue' | 'green';
type Size = 'sm' | 'md' | 'lg';
type ClassName = `${Color}-${Size}`;
// 'red-sm' | 'red-md' | 'red-lg' | 'blue-sm' | ...

// 内置字符串操作类型
type Uppercase<S extends string> = intrinsic;
type Lowercase<S extends string> = intrinsic;
type Capitalize<S extends string> = intrinsic;
type Uncapitalize<S extends string> = intrinsic;

// 实际应用
type EventName<T extends string> = `on${Capitalize<T>}`;
type PropEventName<T extends string> = `${T}Changed`;

type ClickEvent = EventName<'click'>;  // 'onClick'
type NameChanged = PropEventName<'name'>;  // 'nameChanged'
```

### 1.4 模板字面量与联合类型
```typescript
type Vertical = 'top' | 'middle' | 'bottom';
type Horizontal = 'left' | 'center' | 'right';

type Position = `${Vertical}-${Horizontal}`;
// 'top-left' | 'top-center' | 'top-right' | 
// 'middle-left' | 'middle-center' | 'middle-right' | 
// 'bottom-left' | 'bottom-center' | 'bottom-right'

// 解析路径参数
type ExtractParams<T extends string> = 
    T extends `${string}:${infer Param}/${infer Rest}`
        ? Param | ExtractParams<Rest>
        : T extends `${string}:${infer Param}`
            ? Param
            : never;

type Params = ExtractParams<'/users/:userId/posts/:postId'>;
// 'userId' | 'postId'
```

---

## 二、泛型高级用法

### 2.1 泛型约束
```typescript
// 基础约束
type Lengthwise = {
    length: number;
};

function logLength<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

logLength('hello');  // OK
logLength([1, 2, 3]);  // OK
// logLength(123);  // Error

// keyof 约束
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const person = { name: 'John', age: 30 };
getProperty(person, 'name');  // OK
// getProperty(person, 'email');  // Error

// 条件约束
type IsNever<T> = [T] extends [never] ? true : false;
type IsAny<T> = 0 extends (1 & T) ? true : false;
type IsUnknown<T> = IsAny<T] extends true ? false : unknown extends T ? true : false;
```

### 2.2 泛型工具类型
```typescript
// DeepPartial
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object 
        ? T[P] extends Function 
            ? T[P] 
            : DeepPartial<T[P]>
        : T[P];
};

// DeepRequired
type DeepRequired<T> = {
    [P in keyof T]-?: T[P] extends object 
        ? T[P] extends Function 
            ? T[P] 
            : DeepRequired<T[P]>
        : T[P];
};

// DeepNonNullable
type DeepNonNullable<T> = {
    [P in keyof T]: T[P] extends object 
        ? T[P] extends Function 
            ? T[P] 
            : DeepNonNullable<T[P]>
        : NonNullable<T[P]>;
};

// Path 类型
type Path<T> = T extends object 
    ? { [K in keyof T]: K extends string 
        ? T[K] extends object 
            ? `${K}` | `${K}.${Path<T[K]>}`
            : `${K}`
        : never 
    }[keyof T]
    : never;

interface Config {
    db: {
        host: string;
        port: number;
    };
    cache: {
        ttl: number;
    };
}

type ConfigPath = Path<Config>;
// 'db' | 'db.host' | 'db.port' | 'cache' | 'cache.ttl'
```

### 2.3 高级泛型模式
```typescript
// 类型安全的事件发射器
type EventMap = {
    click: { x: number; y: number };
    keydown: { key: string };
    scroll: { top: number; left: number };
};

class TypedEventEmitter<T extends Record<string, any>> {
    private handlers = new Map<keyof T, Set<Function>>();
    
    on<K extends keyof T>(event: K, handler: (data: T[K]) => void): void {
        if (!this.handlers.has(event)) {
            this.handlers.set(event, new Set());
        }
        this.handlers.get(event)!.add(handler);
    }
    
    emit<K extends keyof T>(event: K, data: T[K]): void {
        this.handlers.get(event)?.forEach(handler => handler(data));
    }
    
    off<K extends keyof T>(event: K, handler: (data: T[K]) => void): void {
        this.handlers.get(event)?.delete(handler);
    }
}

const emitter = new TypedEventEmitter<EventMap>();
emitter.on('click', (data) => {
    console.log(data.x, data.y);  // 类型安全
});

// 类型安全的状态管理
type StateUpdater<T> = (prev: T) => T;
type StateSelector<T, R> = (state: T) => R;

class Store<T> {
    private state: T;
    private listeners = new Set<() => void>();
    
    constructor(initialState: T) {
        this.state = initialState;
    }
    
    getState(): T {
        return this.state;
    }
    
    setState(updater: StateUpdater<T>): void {
        this.state = updater(this.state);
        this.listeners.forEach(listener => listener());
    }
    
    subscribe(listener: () => void): () => void {
        this.listeners.add(listener);
        return () => this.listeners.delete(listener);
    }
    
    select<R>(selector: StateSelector<T, R>): R {
        return selector(this.state);
    }
}

// 类型安全的 API 客户端
type APIEndpoints = {
    '/users': {
        GET: { response: User[] };
        POST: { body: CreateUserRequest; response: User };
    };
    '/users/:id': {
        GET: { response: User };
        PUT: { body: UpdateUserRequest; response: User };
        DELETE: { response: void };
    };
};

type ExtractParams<T extends string> = 
    T extends `${string}:${infer Param}/${infer Rest}`
        ? Param | ExtractParams<Rest>
        : T extends `${string}:${infer Param}`
            ? Param
            : never;

type APIResponse<
    Path extends keyof APIEndpoints,
    Method extends keyof APIEndpoints[Path]
> = APIEndpoints[Path][Method] extends { response: infer R } ? R : never;

type UsersResponse = APIResponse<'/users', 'GET'>;  // User[]
```

---

## 三、装饰器

### 3.1 类装饰器
```typescript
// 类装饰器
function Sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

@Sealed
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
}

// 类装饰器工厂
function Logger(prefix: string) {
    return function(constructor: Function) {
        console.log(`${prefix}: ${constructor.name} created`);
    };
}

@Logger('App')
class MyClass {}

// 混入
type Constructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends Constructor>(Base: TBase) {
    return class extends Base {
        timestamp = Date.now();
    };
}

function Activatable<TBase extends Constructor>(Base: TBase) {
    return class extends Base {
        isActive = false;
        activate() { this.isActive = true; }
        deactivate() { this.isActive = false; }
    };
}

class BaseUser {
    name = '';
}

const TimestampedActivatableUser = Timestamped(Activatable(BaseUser));
const user = new TimestampedActivatableUser();
```

### 3.2 方法装饰器
```typescript
// 方法装饰器
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with args:`, args);
        const result = originalMethod.apply(this, args);
        console.log(`${propertyKey} returned:`, result);
        return result;
    };
    
    return descriptor;
}

// 性能测量装饰器
function Measure(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        const start = performance.now();
        const result = originalMethod.apply(this, args);
        const end = performance.now();
        console.log(`${propertyKey} took ${end - start}ms`);
        return result;
    };
    
    return descriptor;
}

// 重试装饰器
function Retry(maxAttempts: number = 3) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalMethod = descriptor.value;
        
        descriptor.value = async function(...args: any[]) {
            let lastError: Error;
            
            for (let attempt = 1; attempt <= maxAttempts; attempt++) {
                try {
                    return await originalMethod.apply(this, args);
                } catch (error) {
                    lastError = error as Error;
                    console.log(`Attempt ${attempt} failed:`, error);
                }
            }
            
            throw lastError!;
        };
        
        return descriptor;
    };
}

// 缓存装饰器
function Cache(ttl: number = 60000) {
    const cache = new Map<string, { value: any; expiry: number }>();
    
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalMethod = descriptor.value;
        
        descriptor.value = function(...args: any[]) {
            const key = `${propertyKey}:${JSON.stringify(args)}`;
            const cached = cache.get(key);
            
            if (cached && cached.expiry > Date.now()) {
                return cached.value;
            }
            
            const result = originalMethod.apply(this, args);
            cache.set(key, { value: result, expiry: Date.now() + ttl });
            
            return result;
        };
        
        return descriptor;
    };
}

class Calculator {
    @Log
    @Measure
    add(a: number, b: number): number {
        return a + b;
    }
    
    @Retry(3)
    async fetchData(url: string): Promise<any> {
        const response = await fetch(url);
        return response.json();
    }
    
    @Cache(5000)
    expensiveOperation(n: number): number {
        console.log('Computing...');
        return Array.from({ length: n }, (_, i) => i).reduce((a, b) => a + b, 0);
    }
}
```

### 3.3 属性装饰器
```typescript
// 属性装饰器
function Validate(validator: (value: any) => boolean) {
    return function(target: any, propertyKey: string) {
        let value: any;
        
        const getter = () => value;
        const setter = (newVal: any) => {
            if (!validator(newVal)) {
                throw new Error(`Invalid value for ${propertyKey}: ${newVal}`);
            }
            value = newVal;
        };
        
        Object.defineProperty(target, propertyKey, {
            get: getter,
            set: setter,
            enumerable: true,
            configurable: true
        });
    };
}

// 必填装饰器
function Required(target: any, propertyKey: string) {
    Object.defineProperty(target, propertyKey, {
        set(value: any) {
            if (value === undefined || value === null) {
                throw new Error(`${propertyKey} is required`);
            }
            // 存储值
            Object.defineProperty(this, propertyKey, {
                value,
                writable: true,
                enumerable: true,
                configurable: true
            });
        },
        enumerable: true,
        configurable: true
    });
}

class UserForm {
    @Required
    name!: string;
    
    @Validate((email) => email.includes('@'))
    email!: string;
    
    @Validate((age) => age >= 0 && age <= 150)
    age!: number;
}
```

### 3.4 参数装饰器
```typescript
// 参数装饰器
function Inject(token: string) {
    return function(target: any, propertyKey: string | symbol, parameterIndex: number) {
        const existingTokens = Reflect.getMetadata('inject:tokens', target, propertyKey) || [];
        existingTokens[parameterIndex] = token;
        Reflect.defineMetadata('inject:tokens', existingTokens, target, propertyKey);
    };
}

// 方法参数验证
function ValidateParam(validator: (value: any) => boolean) {
    return function(target: any, propertyKey: string | symbol, parameterIndex: number) {
        const existingValidators = Reflect.getMetadata('validate:params', target, propertyKey) || [];
        existingValidators[parameterIndex] = validator;
        Reflect.defineMetadata('validate:params', existingValidators, target, propertyKey);
    };
}

class UserService {
    getUser(
        @ValidateParam((id) => typeof id === 'number' && id > 0)
        id: number
    ): User {
        // ...
    }
}
```

---

## 四、类型体操

### 4.1 基础类型体操
```typescript
// 获取函数参数类型
type Parameters<T extends (...args: any) => any> = 
    T extends (...args: infer P) => any ? P : never;

// 获取函数返回类型
type ReturnType<T extends (...args: any) => any> = 
    T extends (...args: any) => infer R ? R : never;

// 获取构造函数参数类型
type ConstructorParameters<T extends abstract new (...args: any) => any> = 
    T extends abstract new (...args: infer P) => any ? P : never;

// 获取实例类型
type InstanceType<T extends abstract new (...args: any) => any> = 
    T extends abstract new (...args: any) => infer R ? R : never;
```

### 4.2 高级类型体操
```typescript
// Deep Readonly
type DeepReadonly<T> = T extends (infer U)[]
    ? ReadonlyArray<DeepReadonly<U>>
    : T extends Map<infer K, infer V>
        ? ReadonlyMap<DeepReadonly<K>, DeepReadonly<V>>
        : T extends Set<infer U>
            ? ReadonlySet<DeepReadonly<U>>
            : T extends object
                ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
                : T;

// Deep Mutable
type DeepMutable<T> = T extends ReadonlyArray<infer U>
    ? U[]
    : T extends ReadonlyMap<infer K, infer V>
        ? Map<DeepMutable<K>, DeepMutable<V>>
        : T extends ReadonlySet<infer U>
            ? Set<DeepMutable<U>>
            : T extends object
                ? { -readonly [K in keyof T]: DeepMutable<T[K]> }
                : T;

// Partial Deep
type PartialDeep<T> = T extends (infer U)[]
    ? PartialDeep<U>[]
    : T extends Map<infer K, infer V>
        ? Map<PartialDeep<K>, PartialDeep<V>>
        : T extends Set<infer U>
            ? Set<PartialDeep<U>>
            : T extends object
                ? { [K in keyof T]?: PartialDeep<T[K]> }
                : T;

// Required Deep
type RequiredDeep<T> = T extends (infer U)[]
    ? RequiredDeep<U>[]
    : T extends Map<infer K, infer V>
        ? Map<RequiredDeep<K>, RequiredDeep<V>>
        : T extends Set<infer U>
            ? Set<RequiredDeep<U>>
            : T extends object
                ? { [K in keyof T]-?: RequiredDeep<T[K]> }
                : T;
```

### 4.3 类型安全的 SQL 查询构建器
```typescript
// 类型安全的查询构建器
type TableName = 'users' | 'posts' | 'comments';

type TableSchema = {
    users: {
        id: number;
        name: string;
        email: string;
        age: number;
    };
    posts: {
        id: number;
        title: string;
        content: string;
        authorId: number;
    };
    comments: {
        id: number;
        content: string;
        postId: number;
        userId: number;
    };
};

type ColumnName<T extends TableName> = keyof TableSchema[T];

type SelectQuery<T extends TableName> = {
    from: T;
    select: ColumnName<T>[];
    where?: Partial<Record<ColumnName<T>, any>>;
    orderBy?: ColumnName<T>;
    limit?: number;
};

class QueryBuilder<T extends TableName> {
    private query: SelectQuery<T> = {} as SelectQuery<T>;
    
    from(table: T): this {
        this.query.from = table;
        return this;
    }
    
    select(...columns: ColumnName<T>[]): this {
        this.query.select = columns;
        return this;
    }
    
    where(conditions: Partial<Record<ColumnName<T>, any>>): this {
        this.query.where = conditions;
        return this;
    }
    
    orderBy(column: ColumnName<T>): this {
        this.query.orderBy = column;
        return this;
    }
    
    limit(n: number): this {
        this.query.limit = n;
        return this;
    }
    
    execute(): Pick<TableSchema[T], ColumnName<T>>[] {
        // 实际执行查询
        return [];
    }
}

// 使用
const qb = new QueryBuilder();
const results = qb
    .from('users')
    .select('id', 'name', 'email')
    .where({ age: 25 })
    .orderBy('name')
    .limit(10)
    .execute();

// results 的类型是 Pick<TableSchema['users'], 'id' | 'name' | 'email'>[]
```

---

## 五、学习资源

| 资源 | 说明 |
|------|------|
| TypeScript 官方文档 | 完整参考 |
| Type Challenges | 类型体操练习 |
| Total TypeScript | 高级 TypeScript 课程 |
| Matt Pocock's Blog | TypeScript 技巧 |

---

*本资料由 AI 整理，建议结合实际项目练习*
