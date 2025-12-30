# TypeScript Interview Questions

## 🔥 Quick Concepts (Less Common Types)

### ⭐ Tuple
**Simple Explanation:** A Tuple is an array with a **fixed number of elements** where **each position has a specific type**.

- Regular Array: Can have any number of items, all same type → `number[]`
- Tuple: Fixed length, each position can have different type → `[string, number]`

```typescript
// Regular array - any number of numbers
let scores: number[] = [10, 20, 30, 40]; // Can add more

// Tuple - exactly 2 items: first is string, second is number
let person: [string, number] = ["John", 25]; // Fixed!

// Accessing tuple
console.log(person[0]); // "John" (string)
console.log(person[1]); // 25 (number)

// Error examples:
// let wrong: [string, number] = [25, "John"];  // ❌ Wrong order
// let wrong2: [string, number] = ["John"];     // ❌ Missing second element
```

**Real-world use:** `useState` in React returns a tuple → `[value, setValue]`

---

### ⭐ Type Inference
**Simple Explanation:** TypeScript is smart! It can **guess the type automatically** from the value you assign.

```typescript
let name = "John";    // TypeScript knows it's a string (no need to write : string)
let age = 25;         // TypeScript knows it's a number
let isActive = true;  // TypeScript knows it's a boolean

// name = 100; // ❌ Error! TypeScript remembers it's a string
```

**Rule:** If TypeScript can figure it out, you don't need to write the type.

---

### ⭐ Basic Types
**Simple Explanation:** The fundamental data types in TypeScript.

| Type | Example | Description |
|------|---------|-------------|
| `string` | `"hello"` | Text |
| `number` | `42`, `3.14` | Numbers (integer or decimal) |
| `boolean` | `true`, `false` | Yes/No values |
| `null` | `null` | Intentionally empty |
| `undefined` | `undefined` | Not yet assigned |
| `any` | anything | Skip type checking (avoid!) |
| `unknown` | anything | Safe "any" - must check before use |

```typescript
let name: string = "John";
let age: number = 25;
let isStudent: boolean = true;
let nothing: null = null;
```

---

### ⭐ Unknown
**Simple Explanation:** `unknown` means "I don't know what type this is yet". You **must check the type before using it**.

```typescript
let value: unknown = "Hello";

// value.toUpperCase(); // ❌ Error! Can't use directly

// ✅ First check, then use
if (typeof value === "string") {
  console.log(value.toUpperCase()); // Now TypeScript knows it's a string
}
```

**Think of it as:** A mystery box 📦 - you must open and check what's inside before using it.

---

### ⭐ Void & Never
**Simple Explanation:**
- `void` = Function **returns nothing** (like `console.log`)
- `never` = Function **never finishes** (throws error or infinite loop)

```typescript
// void - returns nothing
function sayHello(name: string): void {
  console.log("Hello " + name);
  // No return statement
}

// never - never returns (throws error)
function throwError(message: string): never {
  throw new Error(message);
  // Code never reaches here
}

// never - infinite loop
function forever(): never {
  while (true) { }
}
```

**Memory trick:** `void` = empty return, `never` = no return ever!

---

### ⭐ Union Types
**Simple Explanation:** A variable that can be **one of multiple types**. Use `|` (pipe) symbol.

**Why do we need this?**
Normally, a variable holds only **one type** (either `string` OR `number`). But sometimes, based on **business logic**, you may want to store a string sometimes and a number other times in the same variable. Instead of using `any`, we use **Union Types** to define more than one type for a variable.

```typescript
// Normal: variable can only hold ONE type
let name: string = "John";    // Only strings allowed
let age: number = 25;         // Only numbers allowed

// Union Type: variable can hold MULTIPLE types based on business logic
let id: string | number;

id = "ABC123";  // ✅ OK (string) - e.g., User ID from database
id = 123;       // ✅ OK (number) - e.g., Order ID from system
// id = true;   // ❌ Error (boolean not allowed - not in our union)

// Function that accepts multiple types
function printId(id: string | number) {
  console.log("ID: " + id);
}
```

**Think of it as:** "This OR That" → `string | number` means "string OR number"

---

### ⭐ Interfaces
**Simple Explanation:** A **blueprint/contract** that defines what properties an object must have.

```typescript
// Define the shape
interface User {
  name: string;
  age: number;
  email?: string;  // ? means optional
}

// Use the shape
const user: User = {
  name: "John",
  age: 25
  // email is optional, so we can skip it
};

// const badUser: User = { name: "John" }; // ❌ Error! Missing 'age'
```

**Think of it as:** A form template 📝 - defines what fields are required.

---

### ⭐ Types (Type Aliases)
**Simple Explanation:** Create your **own custom type name** and use it for **type safety**. You can restrict a variable to **only accept specific values** - like a whitelist!

**Why do we need this?**
Sometimes you want **tight security** - only certain values are allowed, nothing else. It's like a bus ticket 🎫 - only passengers with valid tickets can board. If someone without a ticket tries to enter, they're blocked!

```typescript
// ===== Example 1: Order Status (Only these values allowed) =====
type OrderStatus = "pending" | "processing" | "shipped" | "delivered";

let status: OrderStatus = "pending";    // ✅ Allowed
status = "shipped";                      // ✅ Allowed
// status = "cancelled";                 // ❌ Error! "cancelled" not in the list

// ===== Example 2: Payment Methods (Restrict to valid options) =====
type PaymentMethod = "credit_card" | "debit_card" | "upi" | "net_banking";

function processPayment(method: PaymentMethod) {
  console.log("Processing via: " + method);
}

processPayment("upi");           // ✅ Valid
// processPayment("cash");       // ❌ Error! Cash not accepted online

// ===== Example 3: User Roles (Security - only valid roles) =====
type UserRole = "admin" | "editor" | "viewer";

let role: UserRole = "admin";    // ✅ Valid role
// role = "superuser";           // ❌ Error! Not a valid role - security!

// ===== Example 4: Traffic Light (Only 3 states possible) =====
type TrafficLight = "red" | "yellow" | "green";

function drive(light: TrafficLight) {
  if (light === "green") console.log("Go!");
  else if (light === "yellow") console.log("Slow down!");
  else console.log("Stop!");
}

// ===== Example 5: API Response Status =====
type ApiStatus = "success" | "error" | "loading";

let apiState: ApiStatus = "loading";
// apiState = "failed";          // ❌ Error! Use "error" instead

// ===== Example 6: Days of Week =====
type Weekday = "Mon" | "Tue" | "Wed" | "Thu" | "Fri";
type Weekend = "Sat" | "Sun";
type Day = Weekday | Weekend;    // Combine types!

let today: Day = "Mon";          // ✅ Any day works
// today = "Holiday";            // ❌ Error! Not a valid day
```

**Think of it as:** 
- 🎫 **Ticket System** - Only valid ticket holders allowed
- 🔒 **Whitelist** - Define exactly what values are permitted
- 🚦 **Traffic Rules** - Only specific states are valid

**Difference from Interface:**
- `type` can define unions, primitives → `type ID = string | number`
- `interface` is only for object shapes

---

### ⭐ Generics
**Simple Explanation:** Write **reusable code** that works with **any type**. Use `<T>` as a placeholder.

**Why do we need this?**
Imagine you have a `print` function that takes a `number` and returns a `number`. But now you also need the same function for `string`. Without generics, you have to **write 2 separate functions**! This is not optimized.

```typescript
// ❌ Problem: Same logic, but different types = Multiple functions!
function printNumber(value: number): number {
  console.log(value);
  return value;
}

function printString(value: string): string {
  console.log(value);
  return value;
}

function printBoolean(value: boolean): boolean {
  console.log(value);
  return value;
}
// This is repetitive and hard to maintain! 😫
```

**✅ Solution: Use Generics - ONE function for ALL types!**

```typescript
// Generic function syntax: functionName<T>(param: T): T
function print<T>(value: T): T {
  console.log(value);
  return value;
}
```

**Two ways to call generic functions:**

```typescript
// Way 1: Explicitly declare the type
print<number>(100);        // T = number, only numbers allowed
print<string>("Hello");    // T = string, only strings allowed
print<boolean>(true);      // T = boolean

// Way 2: Let TypeScript infer the type automatically
print(100);                // TypeScript infers T = number
print("Hello");            // TypeScript infers T = string
print(true);               // TypeScript infers T = boolean
```

**More Examples:**

```typescript
// Generic function to get first element of any array
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

getFirst<number>([1, 2, 3]);      // Explicit: Returns 1 (number)
getFirst<string>(["a", "b"]);     // Explicit: Returns "a" (string)
getFirst([true, false]);          // Inferred: Returns true (boolean)

// Generic function with multiple type parameters
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}

pair<string, number>("age", 25);  // Returns: ["age", 25]
pair("name", "John");              // Inferred: ["name", "John"]
```

**Think of it as:** 
- 🪄 **Magic Box** - Whatever type you put in, the same type comes out! Pass a number → get a number back. Pass a string → get a string back.
- You can adjust generics based on business logic - the function adapts to what you need!

---

### ⭐ Partial
**Simple Explanation:** Makes **all properties optional**. Useful for updates where you change only some fields.

**Two ways to make properties optional:**

### Way 1: Using `?` (Manual - Specific keys only)
```typescript
interface User {
  name: string;       // Required
  age: number;        // Required
  email?: string;     // Optional (with ?)
}

const user1: User = { name: "John", age: 25 };           // ✅ email is optional
const user2: User = { name: "Jane", age: 30, email: "j@j.com" }; // ✅ with email
// const user3: User = { name: "Bob" };                  // ❌ Error! age is required
```

**Problem with `?`:**
- You have to **manually identify** which keys to make optional
- Some fields might be **required for core business logic** (like `name`, `age` during registration)
- But during **update**, you might want to change only one field

### Way 2: Using `Partial<T>` (Automatic - ALL keys become optional)
```typescript
interface User {
  name: string;    // Required in interface
  age: number;     // Required in interface  
  email: string;   // Required in interface
}

// ===== CREATE: All fields required =====
function createUser(user: User) {
  // Must provide name, age, email - all required for registration
}

createUser({ name: "John", age: 25, email: "j@j.com" }); // ✅ All required
// createUser({ name: "John" });  // ❌ Error! Missing age & email

// ===== UPDATE: Use Partial - any field is optional =====
function updateUser(id: number, updates: Partial<User>) {
  // Partial<User> = { name?: string; age?: number; email?: string }
  // All fields become optional automatically!
}

updateUser(1, { name: "John" });              // ✅ Only update name
updateUser(2, { age: 30 });                   // ✅ Only update age
updateUser(3, { email: "new@email.com" });    // ✅ Only update email
updateUser(4, { name: "Jane", age: 28 });     // ✅ Update multiple fields
updateUser(5, {});                            // ✅ Even empty is valid!
```

**When to use what?**

| Scenario | Use | Example |
|----------|-----|---------|
| Some fields always optional | `?` | `email?: string` (user may not have email) |
| All fields required for CREATE | Normal interface | Registration form |
| Any field optional for UPDATE | `Partial<T>` | Edit profile - change only what you want |

**Real-World Example:**
```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  description: string;
  stock: number;
}

// CREATE: All fields required
function addProduct(product: Product) { /* ... */ }

// UPDATE: Change only what you need
function updateProduct(id: number, updates: Partial<Product>) {
  // Maybe just update price: { price: 299 }
  // Maybe just update stock: { stock: 50 }
}

updateProduct(101, { price: 199 });           // ✅ Only price
updateProduct(102, { stock: 0, name: "New" }); // ✅ stock + name
```

**Think of it as:** 
- `?` = "This specific field is always optional"
- `Partial<T>` = "For THIS function, all fields become optional" (like an update form where you change only what you want)

---

### ⭐ Access Modifiers
**Simple Explanation:** Control **who can access** class properties.

| Modifier | Inside Class | Child Class | Outside |
|----------|-------------|-------------|---------|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

**Real-World Analogy - Bank Account:**
- `public` (Account Holder Name) → Everyone can see your name - it's on your card, receipts, etc.
- `protected` (Balance) → Only you and your family members (children) can check balance, not strangers
- `private` (PIN) → Only YOU know your PIN - not even your family should know it!

```typescript
// ===== PARENT CLASS =====
class BankAccount {
  public accountName: string;    // Anyone can see
  protected balance: number;     // Only family (this class + children)
  private pin: number;           // TOP SECRET - only this class

  constructor(name: string, balance: number, pin: number) {
    this.accountName = name;
    this.balance = balance;
    this.pin = pin;
  }

  // Private method - only this class can verify PIN
  private verifyPin(inputPin: number): boolean {
    return this.pin === inputPin;
  }

  // Public method - anyone can call this
  public withdraw(amount: number, inputPin: number): string {
    if (this.verifyPin(inputPin)) {  // ✅ Can access private inside class
      this.balance -= amount;         // ✅ Can access protected inside class
      return "Withdrawn: " + amount;
    }
    return "Wrong PIN!";
  }
}

// ===== CHILD CLASS (extends parent) =====
class SavingsAccount extends BankAccount {
  private interestRate: number = 0.05;

  // Child class CAN access protected members
  addInterest(): void {
    const interest = this.balance * this.interestRate;  // ✅ protected - accessible!
    this.balance += interest;                            // ✅ protected - accessible!
    console.log("New balance: " + this.balance);
  }

  // Child class CANNOT access private members
  showPin(): void {
    // console.log(this.pin);  // ❌ Error! private - NOT accessible in child
  }
}

// ===== OUTSIDE (Using the class) =====
const myAccount = new SavingsAccount("John", 1000, 1234);

// PUBLIC - accessible everywhere ✅
console.log(myAccount.accountName);  // ✅ "John"
myAccount.withdraw(100, 1234);        // ✅ Public method works

// PROTECTED - NOT accessible outside ❌
// console.log(myAccount.balance);    // ❌ Error! Can't access from outside

// PRIVATE - NOT accessible outside ❌
// console.log(myAccount.pin);        // ❌ Error! Can't access from outside
```

**When to use what?**

| Modifier | When to Use | Example |
|----------|-------------|---------|
| `public` | Data that everyone needs to see/use | User's name, product title |
| `protected` | Data that child classes need to inherit & modify | Balance (child can add interest) |
| `private` | Sensitive data that NO ONE else should touch | PIN, password, API keys |

**Think of it as a House 🏠:**
- `public` = Front yard - everyone can see
- `protected` = Living room - only family members allowed
- `private` = Personal diary - only YOU can read

---

### ⭐ Type Guards
**Simple Explanation:** **Check the type at runtime** so TypeScript knows what type you're working with.

**Why do we need Type Guards?**
When you use **Union Types** (like `string | number`), you MUST use Type Guards to avoid runtime errors. Without checking the type, you can't safely use type-specific methods!

```typescript
// If you use like print(3) passing number will cause issue 
function print(value: string | number) {
  console.log(value.toUpperCase());  // ❌ Error! number dont have toUpperCase()
}

// ✅ Solution: Use type guard first
function print(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());  // ✅ Safe - checked it's a string
  } else {
    console.log(value.toFixed(2));     // ✅ Safe - it's a number
  }
}
```

**Real-World Analogy - Security Guard at School Gate 🚪:**

Imagine you appoint a guard at a school gate with these instructions:
- **Boys** → Go to the RIGHT side
- **Girls** → Go to the LEFT side  
- **Teachers** → Go STRAIGHT to the staff room

The guard **checks** each person first, then **directs** them based on their type. You can add more conditions as needed!

```typescript
// ===== Type Guard = Security Guard Checking & Directing =====

type Person = "boy" | "girl" | "teacher";

function directPerson(person: Person) {
  // Guard checks the type and directs accordingly
  if (person === "boy") {
    console.log("Go RIGHT → Boys section");
  } else if (person === "girl") {
    console.log("Go LEFT → Girls section");
  } else {
    console.log("Go STRAIGHT → Staff room");
  }
}

directPerson("boy");      // "Go RIGHT → Boys section"
directPerson("girl");     // "Go LEFT → Girls section"
directPerson("teacher");  // "Go STRAIGHT → Staff room"
```

**More Practical Example:**

```typescript
function process(value: string | number | boolean) {
  // Type Guard checks and allows specific operations
  
  if (typeof value === "string") {
    // Guard says: "It's a string! You can use string methods"
    console.log(value.toUpperCase());  // ✅ String method allowed
    
  } else if (typeof value === "number") {
    // Guard says: "It's a number! You can use number methods"
    console.log(value.toFixed(2));     // ✅ Number method allowed
    
  } else {
    // Guard says: "It's a boolean! Handle accordingly"
    console.log(value ? "Yes" : "No"); // ✅ Boolean logic
  }
}

process("hello");  // "HELLO"
process(3.14159);  // "3.14"
process(true);     // "Yes"
```

**Common Type Guard Methods:**

| Guard | When to Use | Example |
|-------|-------------|---------|
| `typeof` | For primitives (string, number, boolean) | `typeof value === "string"` |
| `instanceof` | For classes/objects | `value instanceof Date` |
| `in` | Check if property exists | `"name" in object` |

```typescript
// typeof - for primitives
if (typeof value === "string") { /* string operations */ }

// instanceof - for classes
if (value instanceof Date) { /* Date operations */ }

// in - for checking property exists
if ("email" in user) { /* user has email property */ }
```

**Think of it as:** 
- 🚪 **Security Guard** - Checks who you are, then directs you to the right place
- Only after the guard confirms your type, you're allowed to do type-specific things

---

### ⭐ Type Assertions
**Simple Explanation:** **Tell TypeScript** "trust me, I know the type". Use `as` keyword.

**Why do we need this?**
Sometimes TypeScript doesn't know the exact type, but YOU know it. Type Assertion is like saying "Don't worry, I'm 100% sure about this!"

**Basic Syntax:**
```typescript
variableName as Type
```

**Easy Examples:**

```typescript
// Example 1: Getting input box from HTML
const box = document.getElementById("username");
const inputBox = document.getElementById("username") as HTMLInputElement;
inputBox.value = "John";

// Example 2: Animal sounds
let pet: any = "dog";
let myDog = pet as string;
console.log(myDog.toUpperCase());

// Example 3: Getting data from API
const apiData: any = { name: "John", age: 25 };
const person = apiData as { name: string; age: number };
console.log(person.name);
console.log(person.age);

// Example 4: Button on webpage
const btn = document.querySelector(".save-btn");
const button = document.querySelector(".save-btn") as HTMLButtonElement;
button.disabled = true;

// Example 5: Number or String
let value: string | number = "100";
let textValue = value as string;
console.log(textValue.toUpperCase());
```

**Real-World Analogy:**
Imagine you ordered a package 📦. The delivery person asks "What's inside?" 

- **Without Type Assertion:** "I don't know, could be anything"
- **With Type Assertion:** "It's a book! Trust me, I ordered it!"

You're telling TypeScript exactly what's in the box so it can help you use it properly.

**Two ways to write it:**

```typescript
// Way 1: Using "as" (Recommended)
let name = value as string;

// Way 2: Using angle brackets
let name = <string>value;
```

**⚠️ Important Warning:**

Type Assertion does NOT change or convert the value! It only tells TypeScript to treat it as that type.

```typescript
let x: any = "hello";
let y = x as number;
console.log(typeof y);  // Still "string"! NOT "number"!
```

**When to use:**

| Situation | Use Type Assertion? |
|-----------|-------------------|
| Getting HTML elements | ✅ Yes |
| You know the data structure | ✅ Yes |
| Don't know the type | ❌ No - Use Type Guards instead |

**Think of it as:** 
- 🎯 **Vouching for someone** - "I guarantee this person is trustworthy!"
- If you're wrong, it's your responsibility!

---

### ⭐ TypeScript with Angular
**Simple Explanation:** Angular uses TypeScript heavily. Key features used:

| Feature | Angular Usage |
|---------|---------------|
| **Decorators** | `@Component`, `@Injectable`, `@Input` |
| **Interfaces** | Define models (`User`, `Product`) |
| **Access Modifiers** | `private` for injected services |
| **Generics** | `Observable<User>`, `HttpClient.get<T>()` |
| **Type Safety** | Typed forms, typed HTTP responses |

```typescript
// Typical Angular component
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html'
})
export class UserComponent {
  // Private service injection
  constructor(private userService: UserService) {}

  // Typed observable
  users$: Observable<User[]> = this.userService.getUsers();
}

// Typed HTTP call
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```

---

<div align="center">

# **📝 INTERVIEW QUESTIONS**

</div>

---

## 1. Why TypeScript?
**Concept:** TypeScript is a strongly-typed superset of JavaScript developed by Microsoft. It compiles down to plain JavaScript and can run anywhere JavaScript runs.

**Key Benefits:**
- **Static Type Checking**: Catches type-related errors at compile time rather than runtime, reducing bugs in production.
- **Enhanced IDE Support**: Provides excellent autocomplete, IntelliSense, navigation, and refactoring capabilities.
- **Self-Documenting Code**: Types serve as inline documentation, making code more readable and maintainable.
- **Modern JavaScript Features**: Supports latest ECMAScript features and compiles them to older versions for browser compatibility.
- **Better Tooling for Large Codebases**: Makes refactoring safer and easier in large applications.
- **Object-Oriented Programming**: Full support for classes, interfaces, inheritance, and access modifiers.

### OOP Concepts Achievable in TypeScript:

| OOP Concept | TypeScript Feature | Description |
|-------------|-------------------|-------------|
| **Classes** | `class` keyword | Blueprint for creating objects with properties and methods |
| **Objects** | `new ClassName()` | Instances created from classes |
| **Encapsulation** | `public`, `private`, `protected` | Hiding internal details using access modifiers |
| **Inheritance** | `extends` | Child class inherits from parent class |
| **Polymorphism** | Method overriding | Same method behaves differently in different classes |
| **Abstraction** | `abstract` class, `interface` | Hide complexity, show only essentials |
| **Interfaces** | `interface`, `implements` | Contract that defines structure without implementation |

```typescript
// Quick OOP Example
// Abstraction & Interface
interface Printable {
  print(): void;
}

// Class, Encapsulation, Abstraction
abstract class Shape {
  abstract getArea(): number;
}

// Inheritance, Polymorphism
class Circle extends Shape implements Printable {
  constructor(private radius: number) { super(); } // Encapsulation (private)
  
  getArea(): number { return Math.PI * this.radius ** 2; } // Polymorphism
  print(): void { console.log(`Area: ${this.getArea()}`); }
}

// Object
const circle = new Circle(5);
circle.print(); // "Area: 78.54..."
```

**Example:**
```typescript
// Without TypeScript (JavaScript)
function add(a, b) {
  return a + b;
}
add("5", 3); // Returns "53" - unexpected string concatenation

// With TypeScript
function add(a: number, b: number): number {
  return a + b;
}
add("5", 3); // Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

---

## 2. Type Inference
**Concept:** TypeScript can automatically infer types based on the values assigned. You don't always need to explicitly annotate types – the compiler is smart enough to figure them out.

**Example:**
```typescript
// TypeScript infers 'message' as string
let message = "Hello, World!";
// message = 42; // Error: Type 'number' is not assignable to type 'string'

// Inferred return type
function multiply(a: number, b: number) {
  return a * b; // Return type inferred as 'number'
}

// Array inference
let numbers = [1, 2, 3]; // Inferred as number[]
let mixed = [1, "hello", true]; // Inferred as (string | number | boolean)[]

// Object inference
let user = {
  name: "Alice",
  age: 30
}; // Inferred as { name: string; age: number }
```

**Best Practice:** Let TypeScript infer types when possible for cleaner code. Add explicit annotations for function parameters, public APIs, and when inference isn't sufficient.

---

## 3. Basic Types
**Concept:** TypeScript provides several basic (primitive) types that form the foundation of the type system.

### Core Primitive Types:
```typescript
// String
let firstName: string = "John";

// Number (integers and floats)
let age: number = 25;
let price: number = 99.99;

// Boolean
let isActive: boolean = true;

// Null and Undefined
let nullValue: null = null;
let undefinedValue: undefined = undefined;

// Symbol
let sym: symbol = Symbol("unique");

// BigInt
let bigNumber: bigint = 100n;
```

### Array Types:
```typescript
// Two syntaxes for arrays
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b", "c"];

// Tuple - fixed length array with specific types at each position
let tuple: [string, number] = ["hello", 42];
```

### Object Type:
```typescript
let person: { name: string; age: number } = {
  name: "Alice",
  age: 30
};
```

---

## 4. `unknown`
**Concept:** The `unknown` type represents any value but is safer than `any`. You cannot perform operations on an `unknown` value without first narrowing it to a more specific type through type checking.

**Example:**
```typescript
let value: unknown = "Hello";

// Cannot use directly - must narrow first
// console.log(value.toUpperCase()); // Error: 'value' is of type 'unknown'

// Type narrowing with typeof
if (typeof value === "string") {
  console.log(value.toUpperCase()); // OK - TypeScript knows it's a string
}

// Type narrowing with instanceof
function processValue(val: unknown) {
  if (val instanceof Date) {
    console.log(val.toISOString()); // OK
  } else if (typeof val === "number") {
    console.log(val.toFixed(2)); // OK
  }
}
```

**Use Case:** Prefer `unknown` over `any` when you don't know the type at compile time but want to maintain type safety.

---

## 5. `any` vs `unknown`
**Concept:** Both can hold any value, but they differ significantly in type safety.

| Feature | `any` | `unknown` |
|---------|-------|-----------|
| Type checking | Disabled | Enabled |
| Property access | Allowed without checks | Requires type narrowing |
| Assignment | Can assign to any type | Can only assign to `any` or `unknown` |
| Safety | Unsafe | Type-safe |

**Example:**
```typescript
// any - Completely bypasses type checking
let anyValue: any = 10;
anyValue.foo.bar.baz; // No compile error (Runtime error!)
anyValue();           // No compile error
let str: string = anyValue; // No error

// unknown - Requires validation before use
let unknownValue: unknown = 10;
// unknownValue.foo; // Error: 'unknownValue' is of type 'unknown'
// unknownValue();   // Error: 'unknownValue' is of type 'unknown'
// let str: string = unknownValue; // Error

// Must narrow the type first
if (typeof unknownValue === "number") {
  console.log(unknownValue.toFixed(2)); // OK
}
```

**Best Practice:** Avoid `any` as it defeats the purpose of TypeScript. Use `unknown` when dealing with dynamic data and validate before use.

---

## 6. `void` & `never`
**Concept:** Both are used for functions but represent different concepts.

### `void`
Represents the absence of a return value. Functions that don't return anything have a `void` return type.

```typescript
function logMessage(message: string): void {
  console.log(message);
  // No return statement or returns undefined
}

// Can also return undefined explicitly
function greet(): void {
  return undefined; // OK
  // return "hello"; // Error
}
```

### `never`
Represents values that never occur. Used for functions that:
- Never return (infinite loops)
- Always throw an error
- Exhaustive type checking

```typescript
// Function that always throws
function throwError(message: string): never {
  throw new Error(message);
}

// Function with infinite loop
function infiniteLoop(): never {
  while (true) {
    // ...
  }
}

// Exhaustive type checking
type Shape = "circle" | "square";

function getArea(shape: Shape): number {
  switch (shape) {
    case "circle":
      return Math.PI * 10 * 10;
    case "square":
      return 10 * 10;
    default:
      // If all cases are handled, this is unreachable
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

**Key Difference:** `void` means "returns nothing useful", `never` means "never returns at all".

---

## 7. Union Types
**Concept:** Union types allow a value to be one of several types. Use the pipe `|` operator to combine types.

**Example:**
```typescript
// Basic union
let id: string | number;
id = "abc123"; // OK
id = 123;      // OK
// id = true;  // Error

// Function with union parameter
function printId(id: string | number): void {
  // Must narrow before type-specific operations
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(2));
  }
}

// Union with null (Nullable types)
function getUser(id: number): User | null {
  // Return user or null if not found
  return users.find(u => u.id === id) || null;
}

// Literal unions (Discriminated unions)
type Status = "pending" | "approved" | "rejected";
let orderStatus: Status = "pending";
// orderStatus = "cancelled"; // Error: not in union
```

---

## 8. Interfaces
**Concept:** Interfaces define the structure (shape) of an object. They describe what properties and methods an object should have.

**Example:**
```typescript
// Basic interface
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Optional property
  readonly createdAt: Date; // Cannot be modified after creation
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
  createdAt: new Date()
};

// Interface with methods
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

// Implementing interface
const calc: Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};

// Index signatures for dynamic keys
interface StringDictionary {
  [key: string]: string;
}

const dict: StringDictionary = {
  hello: "world",
  foo: "bar"
};
```

---

## 9. Types (Type Aliases)
**Concept:** Type aliases create a new name for a type. They can represent primitives, unions, tuples, objects, and more complex types.

**Example:**
```typescript
// Primitive alias
type ID = string | number;

// Object type
type Point = {
  x: number;
  y: number;
};

// Union type alias
type Status = "active" | "inactive" | "pending";

// Tuple type
type Coordinate = [number, number];

// Function type
type Callback = (data: string) => void;

// Complex type with intersection
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type User = {
  id: ID;
  name: string;
};

type TimestampedUser = User & Timestamped; // Intersection

const user: TimestampedUser = {
  id: 1,
  name: "Alice",
  createdAt: new Date(),
  updatedAt: new Date()
};
```

---

## 10. Interface vs Type
**Concept:** Both can define object shapes, but they have important differences.

### Key Differences:

| Feature | Interface | Type |
|---------|-----------|------|
| Declaration merging | ✅ Yes | ❌ No |
| Extends/Implements | ✅ extends keyword | Uses & intersection |
| Primitives/Unions | ❌ Cannot represent | ✅ Can represent |
| Computed properties | ❌ No | ✅ Yes |
| Performance | Slightly better (cached) | Evaluated each time |

**Example:**
```typescript
// Declaration Merging (Only Interface)
interface Window {
  title: string;
}
interface Window {
  size: number;
}
// Window now has both title and size

// Type cannot be merged
type Car = { brand: string };
// type Car = { model: string }; // Error: Duplicate identifier

// Unions (Only Type)
type StringOrNumber = string | number; // Works
// interface StringOrNumber = string | number; // Error

// Extending
interface Animal {
  name: string;
}
interface Dog extends Animal {
  breed: string;
}

// Type intersection (similar to extends)
type AnimalType = { name: string };
type DogType = AnimalType & { breed: string };

// Implementing interfaces in classes
class Labrador implements Dog {
  name = "Max";
  breed = "Labrador";
}
```

**Best Practice:**
- Use **Interface** for object shapes and public APIs (enables declaration merging)
- Use **Type** for unions, primitives, tuples, and complex type transformations

---

## 11. Generics
**Concept:** Generics allow you to write flexible, reusable components that work with any type while maintaining type safety. They are like "type variables" that can be specified when using the component.

**Example:**
```typescript
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

const str = identity<string>("hello"); // T is string
const num = identity(42); // T is inferred as number

// Generic with constraints
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // OK - T must have length
  return arg;
}

logLength("hello");     // OK - string has length
logLength([1, 2, 3]);   // OK - array has length
// logLength(123);      // Error - number has no length

// Multiple type parameters
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}

const p = pair<string, number>("age", 30); // [string, number]

// Generic interface
interface Repository<T> {
  getById(id: number): T;
  getAll(): T[];
  save(item: T): void;
}

// Generic class
class DataStore<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  getAll(): T[] {
    return this.items;
  }
}

const stringStore = new DataStore<string>();
stringStore.add("hello");
```

---

## 12. Utility Types
**Concept:** TypeScript provides built-in utility types to transform types easily.

### Common Utility Types:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Partial<T> - Makes all properties optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; password?: string }

// Required<T> - Makes all properties required
interface OptionalUser {
  id?: number;
  name?: string;
}
type RequiredUser = Required<OptionalUser>;
// { id: number; name: string }

// Readonly<T> - Makes all properties readonly
type ReadonlyUser = Readonly<User>;
const user: ReadonlyUser = { id: 1, name: "John", email: "j@j.com", password: "123" };
// user.name = "Jane"; // Error: Cannot assign to 'name'

// Pick<T, K> - Creates type with only specified properties
type UserCredentials = Pick<User, "email" | "password">;
// { email: string; password: string }

// Omit<T, K> - Creates type without specified properties
type PublicUser = Omit<User, "password">;
// { id: number; name: string; email: string }

// Record<K, T> - Creates object type with keys K and values T
type UserRoles = Record<string, string[]>;
const roles: UserRoles = {
  admin: ["read", "write", "delete"],
  user: ["read"]
};

// Exclude<T, U> - Excludes types from union
type Primitive = string | number | boolean;
type NonBoolean = Exclude<Primitive, boolean>; // string | number

// Extract<T, U> - Extracts types from union
type Extracted = Extract<Primitive, string | number>; // string | number

// NonNullable<T> - Removes null and undefined
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>; // string

// ReturnType<T> - Gets return type of function
function createUser() {
  return { id: 1, name: "John" };
}
type NewUser = ReturnType<typeof createUser>; // { id: number; name: string }

// Parameters<T> - Gets parameter types as tuple
function greet(name: string, age: number): void {}
type GreetParams = Parameters<typeof greet>; // [string, number]
```

---

## 13. Partial
**Concept:** `Partial<T>` makes all properties of type T optional. Extremely useful for update operations where you might only want to modify some fields.

**Example:**
```typescript
interface Todo {
  id: number;
  title: string;
  description: string;
  completed: boolean;
}

// Without Partial - must provide all properties
function createTodo(todo: Todo): Todo {
  return todo;
}

// With Partial - can provide subset of properties
function updateTodo(id: number, updates: Partial<Todo>): Todo {
  const existingTodo = getTodoById(id);
  return { ...existingTodo, ...updates };
}

// Usage
updateTodo(1, { completed: true }); // Only update completed
updateTodo(2, { title: "New Title", description: "New Description" });

// Implementing Partial manually (for understanding)
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};
```

---

## 14. Classes
**Concept:** TypeScript enhances JavaScript classes with type annotations, access modifiers, and additional OOP features.

**Example:**
```typescript
class Person {
  // Properties with types
  name: string;
  private age: number;
  protected id: number;
  readonly birthDate: Date;

  // Constructor
  constructor(name: string, age: number, id: number) {
    this.name = name;
    this.age = age;
    this.id = id;
    this.birthDate = new Date();
  }

  // Methods
  greet(): string {
    return `Hello, I'm ${this.name}`;
  }

  // Getter
  get userAge(): number {
    return this.age;
  }

  // Setter
  set userAge(value: number) {
    if (value > 0) {
      this.age = value;
    }
  }

  // Static method
  static createAnonymous(): Person {
    return new Person("Anonymous", 0, 0);
  }
}

// Parameter properties shorthand
class Employee {
  constructor(
    public name: string,
    private salary: number,
    readonly department: string
  ) {}
  // Properties are automatically created and assigned
}

const emp = new Employee("Alice", 50000, "Engineering");
console.log(emp.name); // "Alice"
// console.log(emp.salary); // Error: private
```

---

## 15. Access Modifiers
**Concept:** TypeScript provides three access modifiers to control visibility of class members.

| Modifier | Class | Subclass | Outside |
|----------|-------|----------|---------|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

**Example:**
```typescript
class BankAccount {
  public accountHolder: string;    // Accessible everywhere
  protected accountNumber: string; // Accessible in class and subclasses
  private balance: number;         // Accessible only in this class

  constructor(holder: string, number: string, initialBalance: number) {
    this.accountHolder = holder;
    this.accountNumber = number;
    this.balance = initialBalance;
  }

  public getBalance(): number {
    return this.balance;
  }

  protected updateBalance(amount: number): void {
    this.balance += amount;
  }
}

class SavingsAccount extends BankAccount {
  private interestRate: number = 0.05;

  addInterest(): void {
    // Can access protected member
    const interest = this.getBalance() * this.interestRate;
    this.updateBalance(interest); // OK - protected

    // Cannot access private member
    // this.balance += interest; // Error: 'balance' is private
  }

  showAccountNumber(): string {
    return this.accountNumber; // OK - protected accessible in subclass
  }
}

const account = new SavingsAccount("John", "123456", 1000);
console.log(account.accountHolder); // OK - public
// console.log(account.accountNumber); // Error - protected
// console.log(account.balance); // Error - private
```

---

## 16. Implements
**Concept:** Classes can implement interfaces to ensure they follow a specific contract. A class must provide implementations for all members defined in the interface.

**Example:**
```typescript
interface Flyable {
  fly(): void;
  altitude: number;
}

interface Swimmable {
  swim(): void;
  depth: number;
}

// Implementing single interface
class Bird implements Flyable {
  altitude: number = 0;

  fly(): void {
    this.altitude = 100;
    console.log("Flying at altitude:", this.altitude);
  }
}

// Implementing multiple interfaces
class Duck implements Flyable, Swimmable {
  altitude: number = 0;
  depth: number = 0;

  fly(): void {
    this.altitude = 50;
    console.log("Duck flying!");
  }

  swim(): void {
    this.depth = 5;
    console.log("Duck swimming!");
  }
}

// Interface extending another interface
interface Vehicle {
  start(): void;
  stop(): void;
}

interface Car extends Vehicle {
  drive(): void;
  wheels: number;
}

class Sedan implements Car {
  wheels: number = 4;

  start(): void {
    console.log("Starting car...");
  }

  stop(): void {
    console.log("Stopping car...");
  }

  drive(): void {
    console.log("Driving...");
  }
}
```

---

## 17. Abstract Classes
**Concept:** Abstract classes are base classes that cannot be instantiated directly. They can contain abstract methods (without implementation) that must be implemented by derived classes, as well as concrete methods (with implementation).

**Example:**
```typescript
abstract class Shape {
  // Abstract property
  abstract name: string;

  // Abstract method - no implementation
  abstract calculateArea(): number;

  // Concrete method - has implementation
  displayArea(): void {
    console.log(`${this.name} area: ${this.calculateArea()}`);
  }
}

class Circle extends Shape {
  name = "Circle";

  constructor(private radius: number) {
    super();
  }

  // Must implement abstract method
  calculateArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  name = "Rectangle";

  constructor(private width: number, private height: number) {
    super();
  }

  calculateArea(): number {
    return this.width * this.height;
  }
}

// const shape = new Shape(); // Error: Cannot create instance of abstract class

const circle = new Circle(5);
circle.displayArea(); // "Circle area: 78.54..."

const rect = new Rectangle(4, 6);
rect.displayArea(); // "Rectangle area: 24"
```

**Abstract vs Interface:**
- Abstract classes can have implemented methods; interfaces cannot
- A class can extend only one abstract class but implement multiple interfaces
- Abstract classes can have constructors; interfaces cannot

---

## 18. Type Guards
**Concept:** Type guards are techniques that narrow a variable's type within a conditional block. They help TypeScript understand the specific type at runtime.

### Types of Type Guards:

```typescript
// 1. typeof guard (for primitives)
function processValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // TypeScript knows it's string
  } else {
    console.log(value.toFixed(2)); // TypeScript knows it's number
  }
}

// 2. instanceof guard (for classes)
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // TypeScript knows it's Dog
  } else {
    animal.meow(); // TypeScript knows it's Cat
  }
}

// 3. in operator guard (for property checking)
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim(); // TypeScript knows it's Fish
  } else {
    animal.fly(); // TypeScript knows it's Bird
  }
}

// 4. User-defined type guard (custom predicate)
interface Admin {
  role: "admin";
  permissions: string[];
}

interface User {
  role: "user";
  email: string;
}

// Type predicate: "user is Admin"
function isAdmin(user: Admin | User): user is Admin {
  return user.role === "admin";
}

function handleUser(user: Admin | User) {
  if (isAdmin(user)) {
    console.log(user.permissions); // TypeScript knows it's Admin
  } else {
    console.log(user.email); // TypeScript knows it's User
  }
}

// 5. Discriminated unions
interface Square {
  kind: "square";
  size: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

type Shape = Square | Rectangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "square":
      return shape.size ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}
```

---

## 19. Type Assertions
**Concept:** Type assertions tell TypeScript to treat a value as a specific type. They don't perform runtime conversion – they only affect compile-time type checking.

### Two Syntaxes:

```typescript
// 1. "as" syntax (recommended)
let value: unknown = "Hello, World!";
let strLength: number = (value as string).length;

// 2. Angle-bracket syntax (not available in JSX/TSX files)
let strLength2: number = (<string>value).length;
```

### Common Use Cases:

```typescript
// Working with DOM elements
const input = document.getElementById("username") as HTMLInputElement;
input.value = "John"; // TypeScript knows it's an input element

// Non-null assertion (when you know value isn't null)
function getValue(arr: number[]): number {
  const value = arr.find(n => n > 5)!; // ! asserts non-null
  return value;
}

// Const assertion (makes literal types)
const config = {
  endpoint: "/api",
  timeout: 5000
} as const;
// config is now { readonly endpoint: "/api"; readonly timeout: 5000 }

// Double assertion (escape hatch - use sparingly)
const value2 = "hello" as unknown as number; // Forces conversion
```

### Type Assertion vs Type Casting:
```typescript
// Type Assertion (TypeScript only - no runtime effect)
let x: any = "hello";
let y = x as number; // Compiles, but x is still "hello" at runtime!
console.log(typeof y); // "string" - NOT "number"!

// Actual conversion (runtime)
let z = Number(x); // Actually converts to number (NaN in this case)
```

**Best Practice:** Use assertions sparingly. Prefer type guards when possible as they provide runtime safety.

---

## 20. TypeScript with Angular
**Concept:** Angular is built with TypeScript and leverages its features extensively. Understanding TypeScript is essential for Angular development.

### Key TypeScript Features Used in Angular:

```typescript
// 1. Decorators
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserComponent implements OnInit {
  // 2. Access modifiers in constructor (Dependency Injection)
  constructor(
    private userService: UserService,
    private router: Router
  ) {}

  // 3. Lifecycle hooks with interfaces
  ngOnInit(): void {
    this.loadUsers();
  }

  // 4. Generics with HTTP
  private loadUsers(): void {
    this.userService.getUsers().subscribe((users: User[]) => {
      this.users = users;
    });
  }
}

// 5. Interfaces for models
interface User {
  id: number;
  name: string;
  email: string;
  role: UserRole;
}

// 6. Enums for constants
enum UserRole {
  Admin = 'ADMIN',
  User = 'USER',
  Guest = 'GUEST'
}

// 7. Type-safe services
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = '/api/users';

  constructor(private http: HttpClient) {}

  // Generics with HttpClient
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  // Partial for updates
  updateUser(id: number, updates: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/${id}`, updates);
  }
}

// 8. Strict template type checking
@Component({
  template: `
    <div *ngFor="let user of users">
      {{ user.name }} <!-- TypeScript checks user has 'name' property -->
    </div>
  `
})
export class UserListComponent {
  users: User[] = [];
}

// 9. Type-safe forms
interface LoginForm {
  email: string;
  password: string;
  rememberMe: boolean;
}

@Component({...})
export class LoginComponent {
  loginForm = new FormGroup<{
    email: FormControl<string>;
    password: FormControl<string>;
    rememberMe: FormControl<boolean>;
  }>({
    email: new FormControl('', { nonNullable: true }),
    password: new FormControl('', { nonNullable: true }),
    rememberMe: new FormControl(false, { nonNullable: true })
  });
}

// 10. Using unknown for external data validation
this.http.get<unknown>('/api/data').pipe(
  map(data => {
    if (isValidUserArray(data)) {
      return data;
    }
    throw new Error('Invalid data format');
  })
);

// Type guard for validation
function isValidUserArray(data: unknown): data is User[] {
  return Array.isArray(data) && 
    data.every(item => 
      typeof item === 'object' && 
      item !== null &&
      'id' in item && 
      'name' in item
    );
}
```

### Angular-Specific TypeScript Patterns:

```typescript
// Generic component with Input
@Component({
  selector: 'app-data-table',
  template: `...`
})
export class DataTableComponent<T> {
  @Input() items: T[] = [];
  @Input() columns: (keyof T)[] = [];
}

// Typed event emitters
@Component({...})
export class SearchComponent {
  @Output() searchChange = new EventEmitter<string>();
  @Output() itemSelected = new EventEmitter<User>();
}

// Typed route parameters
interface RouteParams {
  id: string;
  category: string;
}

this.route.params.subscribe((params: RouteParams) => {
  const id = +params.id; // Convert to number
});

// Typed reactive forms with FormBuilder
interface RegistrationForm {
  username: FormControl<string>;
  email: FormControl<string>;
  age: FormControl<number | null>;
}

this.form = this.fb.group<RegistrationForm>({
  username: this.fb.control('', { nonNullable: true }),
  email: this.fb.control('', { nonNullable: true }),
  age: this.fb.control(null)
});
```

---

## Summary Table

| Topic | Key Takeaway |
|-------|--------------|
| TypeScript | Adds static typing to JavaScript for safer, more maintainable code |
| Type Inference | TypeScript can automatically infer types from values |
| Basic Types | string, number, boolean, null, undefined, symbol, bigint |
| `unknown` | Type-safe version of any, requires narrowing before use |
| `any` vs `unknown` | `any` disables type checking; `unknown` enforces it |
| `void` & `never` | `void` = no return value; `never` = never returns |
| Union Types | Value can be one of several types using `\|` |
| Interfaces | Define object shapes, can be merged, use for contracts |
| Types | Aliases for complex types, unions, primitives |
| Generics | Type variables for reusable, type-safe components |
| Utility Types | Built-in type transformers (Partial, Pick, Omit, etc.) |
| Classes | Enhanced with access modifiers and type annotations |
| Access Modifiers | public, protected, private control visibility |
| Implements | Classes fulfill interface contracts |
| Abstract Classes | Base classes with abstract methods for inheritance |
| Type Guards | Runtime checks that narrow types (typeof, instanceof, in, custom) |
| Type Assertions | Tell TypeScript to treat value as specific type |
| TypeScript + Angular | Decorators, DI, generics, strict templates, typed forms |
