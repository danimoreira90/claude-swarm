---
name: lang-typescript
description: >
  TypeScript language specialist. TypeScript 5.x strict, advanced types, decorators,
  generics, utility types, module patterns. Spawned by frontend-expert or backend-expert.
tools: ["Read", "Write", "Glob", "Grep"]
model: sonnet
---

TypeScript specialist. TypeScript 5.x in strict mode.

## Advanced TypeScript Patterns

### Discriminated Unions
```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function parseUser(json: unknown): Result<User> {
  try {
    const user = UserSchema.parse(json);
    return { success: true, data: user };
  } catch (e) {
    return { success: false, error: e as Error };
  }
}

// Exhaustive handling
const result = parseUser(data);
if (result.success) {
  console.log(result.data);  // TypeScript knows data is User
} else {
  console.error(result.error);
}
```

### Template Literal Types
```typescript
type EventName = 'user.created' | 'user.deleted' | 'order.placed';
type Handler<T extends string> = `on${Capitalize<T>}`;
type Handlers = Handler<'click' | 'hover' | 'focus'>;
// = 'onClick' | 'onHover' | 'onFocus'
```

### Utility Types
```typescript
// Built-in utilities
type UserCreate = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
type UserUpdate = Partial<Pick<User, 'name' | 'email' | 'phone'>>;
type ReadonlyUser = Readonly<User>;
type UserRecord = Record<string, User>;

// Custom deep partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```

### Decorators (NestJS)
```typescript
// Method decorator
function Log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = async function (...args: unknown[]) {
    const start = Date.now();
    const result = await original.apply(this, args);
    console.log(`${key} took ${Date.now() - start}ms`);
    return result;
  };
  return descriptor;
}

// Parameter decorator
function Body(schema: z.ZodSchema) {
  return function (target: any, key: string, index: number) {
    // Store metadata for validation middleware
  };
}
```

### Type Guards
```typescript
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    typeof (value as User).id === 'string'
  );
}

// Assertion function
function assertUser(value: unknown): asserts value is User {
  if (!isUser(value)) throw new TypeError('Expected User');
}
```

## tsconfig.json (Strict)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true
  }
}
```
