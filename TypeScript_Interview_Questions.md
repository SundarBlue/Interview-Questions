# TypeScript Interview Questions

## 1. Why TypeScript?
**Concept:** TypeScript is a superset of JavaScript that adds static typing.
**Benefits:**
- **Compile-time error checking**: Catches bugs before running code.
- **Better Tooling**: Autocomplete, navigation, refactoring.
- **Readability**: Types serve as documentation.
- **Modern JS features**: Compiles down to older JS versions.

## 2. `any` vs `unknown`
**Concept:**
- **`any`**: Disables type checking. You can access any property or method. Unsafe.
- **`unknown`**: A type-safe counterpart to `any`. You cannot perform operations on it without narrowing the type first.

**Example:**
```typescript
let valAny: any = 10;
valAny.foo(); // No error at compile time (Runtime error)

let valUnknown: unknown = 10;
// valUnknown.foo(); // Error: Object is of type 'unknown'.

if (typeof valUnknown === 'number') {
  console.log(valUnknown.toFixed(2)); // OK
}
```

## 3. Interface vs Type
**Concept:** Both define shapes of objects, but have subtle differences.
- **Interface**:
  - Can be merged (Declaration Merging).
  - Better for defining object shapes and contracts for classes (`implements`).
- **Type**:
  - Can define unions, primitives, tuples, and intersections.
  - Cannot be merged.

**Example:**
```typescript
// Interface
interface User {
  name: string;
}
interface User { // Merges
  age: number;
}

// Type
type ID = string | number; // Union

// Intersection Type
type Draggable = { drag: () => void };
type Resizable = { resize: () => void };
type UIWidget = Draggable & Resizable; // Intersection

const widget: UIWidget = {
  drag: () => {},
  resize: () => {}
};
```

## 4. Generics
**Concept:** Allows creating reusable components that work with a variety of types rather than a single one.

**Example:**
```typescript
function identity<T>(arg: T): T {
  return arg;
}

let output = identity<string>("myString");
let numOutput = identity<number>(100);
```

## 5. Utility Types (Partial, etc.)
**Concept:** TypeScript provides global utilities to transform types.
- **`Partial<T>`**: Makes all properties in T optional.
- **`Pick<T, K>`**: Constructs a type by picking set of properties K from T.
- **`Omit<T, K>`**: Constructs a type by picking all properties from T and then removing K.

**Example:**
```typescript
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

// Pick
type TodoPreview = Pick<Todo, "title">;
const preview: TodoPreview = { title: "Clean room" };

// Omit
type TodoInfo = Omit<Todo, "description">;
const info: TodoInfo = { title: "Clean room" };
```

## 6. Type vs Interface vs Generics (Summary)
**Concept:**
- **Type**: Best for unions, intersections, primitives. Cannot be re-opened (merged).
- **Interface**: Best for object shapes, class contracts (`implements`). Can be merged.
- **Generics**: "Variables for types". Allows functions/classes to work with any type while maintaining type safety.

**Comparison:**
```typescript
// Generic Interface
interface Box<T> {
  contents: T;
}

// Generic Type
type BoxType<T> = {
  contents: T;
};

const stringBox: Box<string> = { contents: "Hello" };
```

## 7. Type Guards
**Concept:** Expressions that perform a runtime check that guarantees the type in some scope.
- `typeof`
- `instanceof`
- User-defined type guards (`arg is Type`)

**Example:**
```typescript
function isString(test: any): test is string {
  return typeof test === "string";
}

function example(x: number | string) {
  if (isString(x)) {
    console.log(x.toUpperCase()); // TS knows x is string here
  }
}

// instanceof Example
class Dog { bark() {} }
class Cat { meow() {} }

function interact(pet: Dog | Cat) {
  if (pet instanceof Dog) {
    pet.bark();
  } else {
    pet.meow();
  }
}
```
