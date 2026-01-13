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

### ⭐ Enums
**Simple Explanation:** Enums allow you to define a **set of named constants**. Think of it as a collection of related values that are more readable than using raw numbers or strings.

**Why do we need this?**
Instead of using magic numbers (0, 1, 2) or strings that can have typos, enums provide **meaningful names** and **autocomplete support**.

```typescript
// ===== Numeric Enums (Default) =====
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right    // 3
}

let move: Direction = Direction.Up;
console.log(move);  // 0

// Can set starting value
enum Status {
  Pending = 1,
  Approved,    // 2
  Rejected     // 3
}

// ===== String Enums (Recommended for APIs) =====
enum OrderStatus {
  Pending = "PENDING",
  Processing = "PROCESSING",
  Shipped = "SHIPPED",
  Delivered = "DELIVERED"
}

let status: OrderStatus = OrderStatus.Shipped;
console.log(status);  // "SHIPPED"

// ===== Const Enums (Optimized - no extra code generated) =====
const enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE"
}

let favoriteColor = Color.Red;  // Replaced with "RED" at compile time

// ===== Heterogeneous Enums (Mixed - Not recommended) =====
enum Mixed {
  No = 0,
  Yes = "YES"
}

// ===== Computed Enums =====
enum FileAccess {
  None = 0,
  Read = 1 << 1,      // 2
  Write = 1 << 2,     // 4
  ReadWrite = Read | Write  // 6
}
```

**Real-World Example:**
```typescript
enum HttpStatus {
  OK = 200,
  BadRequest = 400,
  Unauthorized = 401,
  NotFound = 404,
  InternalServerError = 500
}

function handleResponse(status: HttpStatus) {
  switch (status) {
    case HttpStatus.OK:
      console.log("Success!");
      break;
    case HttpStatus.NotFound:
      console.log("Resource not found");
      break;
    default:
      console.log("Error occurred");
  }
}

handleResponse(HttpStatus.OK);
```

**Think of it as:** 
- 📋 **Menu with fixed options** - Instead of remembering numbers, you select from a named list
- Prevents typos and provides IntelliSense support

---

### ⭐ Intersection Types
**Simple Explanation:** Combine multiple types into **one type that has ALL properties** from each type. Use `&` (ampersand).

**Difference from Union:**
- **Union (`|`)** = "This OR That" (can be one of the types)
- **Intersection (`&`)** = "This AND That" (must have all properties)

```typescript
// ===== Basic Intersection =====
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: number;
  department: string;
};

// Intersection combines both
type EmployeePerson = Person & Employee;

const emp: EmployeePerson = {
  name: "John",
  age: 30,
  employeeId: 12345,
  department: "IT"
  // Must have ALL properties from Person AND Employee
};

// ===== Practical Example - Adding Timestamps =====
type Timestamps = {
  createdAt: Date;
  updatedAt: Date;
};

type Product = {
  id: number;
  name: string;
  price: number;
};

type ProductWithTimestamps = Product & Timestamps;

const product: ProductWithTimestamps = {
  id: 1,
  name: "Laptop",
  price: 999,
  createdAt: new Date(),
  updatedAt: new Date()
};

// ===== Mixing with Types =====
type Draggable = {
  drag: () => void;
};

type Resizable = {
  resize: () => void;
};

type UIWidget = Draggable & Resizable;

const widget: UIWidget = {
  drag: () => console.log("Dragging"),
  resize: () => console.log("Resizing")
};
```

**Think of it as:** 
- 🧩 **Puzzle pieces** - Each piece adds more properties, final object has all pieces combined
- Like inheriting from multiple sources

---

### ⭐ Discriminated Unions
**Simple Explanation:** A pattern where you use a **common property (discriminator)** to distinguish between different types in a union. Makes TypeScript smart enough to know which type you're working with.

**Why do we need this?**
When you have similar objects with slight differences, discriminated unions help TypeScript **narrow down the exact type** based on a common field.

```typescript
// ===== Payment Methods Example =====
interface CreditCardPayment {
  kind: "credit";           // Discriminator
  cardNumber: string;
  cvv: string;
}

interface PayPalPayment {
  kind: "paypal";           // Discriminator
  email: string;
}

interface CashPayment {
  kind: "cash";             // Discriminator
  amount: number;
}

type Payment = CreditCardPayment | PayPalPayment | CashPayment;

function processPayment(payment: Payment) {
  // TypeScript knows which properties exist based on 'kind'
  switch (payment.kind) {
    case "credit":
      console.log("Processing card:", payment.cardNumber);  // ✅ TypeScript knows cardNumber exists
      break;
    case "paypal":
      console.log("Processing PayPal:", payment.email);     // ✅ TypeScript knows email exists
      break;
    case "cash":
      console.log("Received cash:", payment.amount);        // ✅ TypeScript knows amount exists
      break;
  }
}

// ===== API Response Example =====
interface SuccessResponse {
  status: "success";
  data: any;
}

interface ErrorResponse {
  status: "error";
  message: string;
  errorCode: number;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  if (response.status === "success") {
    console.log("Data:", response.data);      // ✅ TypeScript knows data exists
  } else {
    console.log("Error:", response.message);  // ✅ TypeScript knows message exists
  }
}

// ===== Shape Example =====
interface Circle {
  type: "circle";
  radius: number;
}

interface Rectangle {
  type: "rectangle";
  width: number;
  height: number;
}

interface Triangle {
  type: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;

function calculateArea(shape: Shape): number {
  switch (shape.type) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      // Exhaustiveness check
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

**Think of it as:** 
- 🏷️ **Label on a package** - The 'kind' or 'type' field tells you what's inside, so you know how to handle it
- Smart switch that knows exactly what properties are available

---

### ⭐ Keyof Operator
**Simple Explanation:** `keyof` gets all the **property names (keys)** of a type as a union of string literals.

```typescript
// ===== Basic keyof =====
interface User {
  id: number;
  name: string;
  email: string;
}

type UserKeys = keyof User;  // "id" | "name" | "email"

let key: UserKeys;
key = "id";     // ✅ OK
key = "name";   // ✅ OK
// key = "age"; // ❌ Error - not a key of User

// ===== Generic function with keyof =====
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = { id: 1, name: "John", email: "john@example.com" };

const userName = getProperty(user, "name");    // Type: string
const userId = getProperty(user, "id");        // Type: number
// const age = getProperty(user, "age");       // ❌ Error - 'age' not in User

// ===== Practical Example - Type-safe updater =====
function updateProperty<T, K extends keyof T>(
  obj: T,
  key: K,
  value: T[K]
): void {
  obj[key] = value;
}

const product = { id: 1, name: "Laptop", price: 999 };
updateProperty(product, "price", 899);           // ✅ OK
// updateProperty(product, "price", "cheap");    // ❌ Error - must be number
// updateProperty(product, "color", "red");      // ❌ Error - 'color' doesn't exist

// ===== With Mapped Types =====
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// {
//   getId: () => number;
//   getName: () => string;
//   getEmail: () => string;
// }
```

**Think of it as:** 
- 🔑 **Getting all key names from an object** - Creates a type that can only be one of the existing keys

---

### ⭐ Typeof Operator
**Simple Explanation:** The `typeof` operator gets the **type of a variable or property**. Works at the type level, not runtime.

```typescript
// ===== Basic typeof =====
const person = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

type Person = typeof person;
// { name: string; age: number; email: string }

// ===== With functions =====
function greet(name: string, age: number) {
  return `Hello ${name}, you are ${age} years old`;
}

type GreetFunction = typeof greet;
// (name: string, age: number) => string

type GreetParams = Parameters<typeof greet>;
// [string, number]

type GreetReturn = ReturnType<typeof greet>;
// string

// ===== With const =====
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
} as const;

type Config = typeof config;
// {
//   readonly apiUrl: "https://api.example.com";
//   readonly timeout: 5000;
//   readonly retries: 3;
// }

// ===== Practical Example - Reuse existing object structure =====
const defaultSettings = {
  theme: "dark",
  fontSize: 14,
  autoSave: true
};

// Create type from existing object
type Settings = typeof defaultSettings;

function updateSettings(settings: Settings) {
  // settings matches the structure of defaultSettings
}

// ===== With Enums =====
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE"
}

type ColorEnum = typeof Color;
// {
//   Red: Color.Red;
//   Green: Color.Green;
//   Blue: Color.Blue;
// }

const colors: ColorEnum = Color;
```

**Think of it as:** 
- 📸 **Taking a snapshot** - Captures the type of an existing value
- Useful for inferring types from runtime values

---

### ⭐ Indexed Access Types
**Simple Explanation:** Access the **type of a specific property** using bracket notation.

```typescript
// ===== Basic Indexed Access =====
interface User {
  id: number;
  name: string;
  email: string;
  address: {
    street: string;
    city: string;
  };
}

type UserId = User["id"];           // number
type UserName = User["name"];       // string
type UserAddress = User["address"]; // { street: string; city: string }

// ===== Nested Access =====
type City = User["address"]["city"]; // string

// ===== Array Element Type =====
type StringArray = string[];
type ArrayElement = StringArray[number]; // string

const users: User[] = [];
type UserFromArray = typeof users[number]; // User

// ===== Multiple Keys =====
type UserContactInfo = User["name" | "email"];
// string (union of name and email types, both are strings)

type UserIdOrName = User["id" | "name"];
// string | number

// ===== Practical Example - Extract nested types =====
interface ApiResponse {
  data: {
    users: Array<{
      id: number;
      profile: {
        firstName: string;
        lastName: string;
      };
    }>;
  };
}

type UserData = ApiResponse["data"]["users"][number];
// { id: number; profile: { firstName: string; lastName: string } }

type UserProfile = UserData["profile"];
// { firstName: string; lastName: string }

// ===== With Generics =====
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
  address: { street: "123 Main St", city: "NYC" }
};

const email = getProperty(user, "email"); // Type: string
```

**Think of it as:** 
- 🎯 **Drilling down** - Access specific property types from complex types
- Like using dot notation but at the type level

---

### ⭐ Mapped Types
**Simple Explanation:** Create new types by **transforming properties** of an existing type. Like a loop over type properties.

```typescript
// ===== Basic Mapped Type =====
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string }

// ===== Optional Mapped Type =====
type Optional<T> = {
  [K in keyof T]?: T[K];
};

type OptionalUser = Optional<User>;
// { id?: number; name?: string }

// ===== Nullable Mapped Type =====
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type NullableUser = Nullable<User>;
// { id: number | null; name: string | null }

// ===== Adding Prefix/Suffix to Keys =====
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// {
//   getId: () => number;
//   getName: () => string;
// }

// ===== Filtering Properties =====
type RemoveReadonly<T> = {
  -readonly [K in keyof T]: T[K];
};

type RemoveOptional<T> = {
  [K in keyof T]-?: T[K];
};

// ===== Conditional Mapped Types =====
type StringProperties<T> = {
  [K in keyof T]: T[K] extends string ? T[K] : never;
};

interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
}

type ProductStrings = StringProperties<Product>;
// { id: never; name: string; description: string; price: never }

// ===== Practical Example - API Response Mapper =====
type ApiModel<T> = {
  [K in keyof T]: {
    value: T[K];
    loading: boolean;
    error: string | null;
  };
};

type UserApiModel = ApiModel<User>;
// {
//   id: { value: number; loading: boolean; error: string | null };
//   name: { value: string; loading: boolean; error: string | null };
// }
```

**Think of it as:** 
- 🔄 **Transform machine** - Takes a type and transforms each property according to rules
- Like Array.map() but for types

---

### ⭐ Conditional Types
**Simple Explanation:** Types that **choose between two types based on a condition**. Uses `extends` keyword like a ternary operator.

**Syntax:**
```typescript
T extends U ? X : Y
// If T is assignable to U, then type is X, else type is Y
```

**Examples:**

```typescript
// ===== Basic Conditional Type =====
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>;   // "yes"
type B = IsString<number>;   // "no"

// ===== Extract Non-Nullable =====
type NonNullable<T> = T extends null | undefined ? never : T;

type C = NonNullable<string | null>;      // string
type D = NonNullable<number | undefined>; // number

// ===== Function Return Type Helper =====
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string }

// ===== Array Element Type =====
type ElementType<T> = T extends (infer U)[] ? U : T;

type E = ElementType<string[]>;  // string
type F = ElementType<number>;    // number

// ===== Practical Example - API Response Type =====
type ApiResponse<T> = T extends { data: infer D } ? D : T;

type Response1 = ApiResponse<{ data: { id: number } }>;  // { id: number }
type Response2 = ApiResponse<{ id: number }>;            // { id: number }

// ===== Exclude/Extract Types =====
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;

type G = Exclude<"a" | "b" | "c", "a">;        // "b" | "c"
type H = Extract<"a" | "b" | "c", "a" | "b">;  // "a" | "b"

// ===== Flatten Nested Arrays =====
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;

type I = Flatten<string[]>;        // string
type J = Flatten<string[][]>;      // string
type K = Flatten<string[][][]>;    // string

// ===== Distributive Conditional Types =====
type ToArray<T> = T extends any ? T[] : never;

type L = ToArray<string | number>;  // string[] | number[]
```

**Think of it as:** 
- 🎲 **If-else for types** - Choose type based on condition
- Extremely powerful for creating flexible, reusable type utilities

---

### ⭐ Template Literal Types
**Simple Explanation:** Build types using **template string syntax**. Combine string literal types in creative ways.

```typescript
// ===== Basic Template Literal =====
type World = "world";
type Greeting = `hello ${World}`;  // "hello world"

// ===== Event Names =====
type EventName = "click" | "scroll" | "mousemove";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onScroll" | "onMousemove"

// ===== CRUD Operations =====
type Entity = "user" | "product" | "order";
type Action = "create" | "read" | "update" | "delete";
type Permission = `${Action}_${Entity}`;
// "create_user" | "read_user" | "update_user" | "delete_user"
// | "create_product" | "read_product" | ... (12 combinations)

// ===== HTTP Methods with Routes =====
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Route = "/users" | "/products";
type Endpoint = `${HttpMethod} ${Route}`;
// "GET /users" | "POST /users" | "PUT /users" | "DELETE /users"
// | "GET /products" | "POST /products" | ...

// ===== Practical Example - Type-safe Event Emitter =====
type Events = {
  userCreated: { id: number; name: string };
  userDeleted: { id: number };
  orderPlaced: { orderId: string; total: number };
};

type EventKeys = keyof Events;
// "userCreated" | "userDeleted" | "orderPlaced"

type OnEvent = `on${Capitalize<EventKeys>}`;
// "onUserCreated" | "onUserDeleted" | "onOrderPlaced"

class EventEmitter {
  onUserCreated(callback: (data: Events["userCreated"]) => void) {}
  onUserDeleted(callback: (data: Events["userDeleted"]) => void) {}
  onOrderPlaced(callback: (data: Events["orderPlaced"]) => void) {}
}

// ===== CSS Properties =====
type Color = "red" | "green" | "blue";
type Size = "sm" | "md" | "lg";
type ClassName = `${Color}-${Size}`;
// "red-sm" | "red-md" | "red-lg" | "green-sm" | ...

// ===== API Versioning =====
type Version = "v1" | "v2" | "v3";
type Resource = "users" | "posts";
type ApiPath = `/api/${Version}/${Resource}`;
// "/api/v1/users" | "/api/v1/posts" | "/api/v2/users" | ...

// ===== Intrinsic String Manipulation Types =====
type Uppercase<S extends string> = intrinsic;
type Lowercase<S extends string> = intrinsic;
type Capitalize<S extends string> = intrinsic;
type Uncapitalize<S extends string> = intrinsic;

type A = Uppercase<"hello">;      // "HELLO"
type B = Lowercase<"WORLD">;      // "world"
type C = Capitalize<"typescript">; // "Typescript"
type D = Uncapitalize<"Hello">;   // "hello"
```

**Think of it as:** 
- 🎨 **String templates for types** - Build complex string literal types from combinations
- Great for creating type-safe APIs with string-based keys

---

### ⭐ Function Overloads
**Simple Explanation:** Define **multiple function signatures** for the same function name. TypeScript chooses the right signature based on arguments.

**Why do we need this?**
Sometimes a function can accept different types of arguments and return different types based on the input. Overloads make this type-safe.

```typescript
// ===== Basic Overload =====
// Overload signatures
function getValue(id: number): string;
function getValue(name: string): number;

// Implementation signature (must be compatible with all overloads)
function getValue(param: number | string): string | number {
  if (typeof param === "number") {
    return `ID: ${param}`;
  } else {
    return param.length;
  }
}

const result1 = getValue(123);     // Type: string
const result2 = getValue("John");  // Type: number

// ===== Constructor Overload =====
class User {
  name: string;
  age: number;

  // Overload signatures
  constructor(name: string);
  constructor(name: string, age: number);

  // Implementation
  constructor(name: string, age?: number) {
    this.name = name;
    this.age = age ?? 0;
  }
}

const user1 = new User("John");
const user2 = new User("Jane", 25);

// ===== Practical Example - Flexible API =====
interface Contact {
  id: number;
  name: string;
  email: string;
}

// Overload signatures
function findContact(id: number): Contact;
function findContact(email: string): Contact;
function findContact(searchObj: { name: string }): Contact[];

// Implementation
function findContact(
  param: number | string | { name: string }
): Contact | Contact[] {
  if (typeof param === "number") {
    // Find by ID
    return { id: param, name: "John", email: "john@example.com" };
  } else if (typeof param === "string") {
    // Find by email
    return { id: 1, name: "John", email: param };
  } else {
    // Find by name (returns array)
    return [{ id: 1, name: param.name, email: "john@example.com" }];
  }
}

const contact1 = findContact(1);              // Type: Contact
const contact2 = findContact("j@j.com");      // Type: Contact
const contacts = findContact({ name: "John" }); // Type: Contact[]

// ===== Method Overload in Class =====
class Calculator {
  // Overloads
  add(a: number, b: number): number;
  add(a: string, b: string): string;
  add(a: number[], b: number[]): number[];

  // Implementation
  add(a: any, b: any): any {
    if (typeof a === "number" && typeof b === "number") {
      return a + b;
    } else if (typeof a === "string" && typeof b === "string") {
      return a + b;
    } else if (Array.isArray(a) && Array.isArray(b)) {
      return [...a, ...b];
    }
  }
}

const calc = new Calculator();
const num = calc.add(1, 2);           // Type: number
const str = calc.add("Hello", " ");   // Type: string
const arr = calc.add([1], [2, 3]);    // Type: number[]

// ===== With Generics =====
function process<T extends string>(value: T): T;
function process<T extends number>(value: T): T;
function process(value: any): any {
  return value;
}
```

**Think of it as:** 
- 📞 **Same phone number, different departments** - One function name, but behaves differently based on what you pass
- Better than using union types when return types differ based on input

---

### ⭐ Decorators
**Simple Explanation:** Decorators are **special functions** that can modify or annotate classes, methods, properties, or parameters. They use `@` syntax and are widely used in Angular.

**Note:** Enable decorators in `tsconfig.json`:
```json
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

```typescript
// ===== Class Decorator =====
function Component(target: Function) {
  console.log("Component decorator called");
  target.prototype.componentId = Math.random();
}

@Component
class MyComponent {
  // This class now has componentId property
}

// ===== Method Decorator =====
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @Log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// Output:
// Calling add with [2, 3]
// Result: 5

// ===== Property Decorator =====
function MinLength(length: number) {
  return function (target: any, propertyKey: string) {
    let value: string;

    const getter = () => value;
    const setter = (newVal: string) => {
      if (newVal.length < length) {
        throw new Error(`${propertyKey} must be at least ${length} characters`);
      }
      value = newVal;
    };

    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter
    });
  };
}

class User {
  @MinLength(3)
  username: string;

  constructor(username: string) {
    this.username = username;
  }
}

// const user = new User("ab"); // Error: username must be at least 3 characters
const user = new User("john"); // ✅ OK

// ===== Parameter Decorator =====
function Required(target: any, propertyKey: string, parameterIndex: number) {
  console.log(`Parameter in ${propertyKey} at index ${parameterIndex} is required`);
}

class ProductService {
  getProduct(@Required id: number): void {
    console.log(`Getting product ${id}`);
  }
}

// ===== Decorator Factory (with parameters) =====
function Authorize(role: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      console.log(`Checking if user has role: ${role}`);
      // Authorization logic here
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class AdminController {
  @Authorize("admin")
  deleteUser(userId: number): void {
    console.log(`Deleting user ${userId}`);
  }

  @Authorize("moderator")
  banUser(userId: number): void {
    console.log(`Banning user ${userId}`);
  }
}

// ===== Multiple Decorators =====
function First() {
  console.log("First decorator factory");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("First decorator called");
  };
}

function Second() {
  console.log("Second decorator factory");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("Second decorator called");
  };
}

class Example {
  @First()
  @Second()
  method() {}
}
// Output:
// First decorator factory
// Second decorator factory
// Second decorator called
// First decorator called

// ===== Real-World Example - Angular-like =====
function Injectable() {
  return function <T extends { new (...args: any[]): {} }>(constructor: T) {
    // Mark class as injectable
    return class extends constructor {
      injected = true;
    };
  };
}

@Injectable()
class UserService {
  getUsers() {
    return ["John", "Jane"];
  }
}
```

**Think of it as:** 
- 🏷️ **Sticky notes with instructions** - Attach metadata or behavior to classes/methods
- Used heavily in frameworks like Angular for dependency injection, routing, etc.

---

### ⭐ Modules & Namespaces
**Simple Explanation:** Organize code into reusable pieces. **Modules** (ES6) are preferred; **Namespaces** (older TypeScript feature) are less common now.

```typescript
// ===== ES6 Modules (Recommended) =====

// --- math.ts ---
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export const PI = 3.14159;

export default class Calculator {
  multiply(a: number, b: number): number {
    return a * b;
  }
}

// --- app.ts ---
// Named imports
import { add, subtract, PI } from "./math";

// Default import
import Calculator from "./math";

// Rename imports
import { add as sum } from "./math";

// Import all
import * as MathUtils from "./math";

console.log(add(2, 3));              // 5
console.log(MathUtils.PI);           // 3.14159
const calc = new Calculator();
console.log(calc.multiply(2, 3));    // 6

// ===== Namespaces (Older TypeScript approach) =====

// --- utils.ts ---
namespace StringUtils {
  export function capitalize(str: string): string {
    return str.charAt(0).toUpperCase() + str.slice(1);
  }

  export function reverse(str: string): string {
    return str.split("").reverse().join("");
  }

  // Not exported - private to namespace
  function helper() {
    return "helper";
  }
}

// Usage
const result = StringUtils.capitalize("hello"); // "Hello"

// ===== Nested Namespaces =====
namespace App {
  export namespace Models {
    export interface User {
      id: number;
      name: string;
    }

    export interface Product {
      id: number;
      name: string;
    }
  }

  export namespace Services {
    export class UserService {
      getUser(id: number): Models.User {
        return { id, name: "John" };
      }
    }
  }
}

const userService = new App.Services.UserService();
const user: App.Models.User = userService.getUser(1);

// ===== Merging Namespaces =====
namespace Animals {
  export class Dog {
    bark() { console.log("Woof!"); }
  }
}

namespace Animals {
  export class Cat {
    meow() { console.log("Meow!"); }
  }
}

// Both Dog and Cat are now in Animals namespace
const dog = new Animals.Dog();
const cat = new Animals.Cat();

// ===== Module Augmentation (Extending existing modules) =====
// Extend existing module
declare module "express" {
  interface Request {
    user?: {
      id: number;
      name: string;
    };
  }
}

// Now Express Request has user property
```

**Think of it as:** 
- 📦 **Packaging code** - Modules = separate files, Namespaces = logical grouping
- **Prefer ES6 modules** for modern TypeScript/JavaScript

---

### ⭐ Type-Safe Async/Await & Promises
**Simple Explanation:** Make asynchronous operations type-safe with proper typing for Promises and async functions.

```typescript
// ===== Basic Promise Types =====
function fetchUser(): Promise<{ id: number; name: string }> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ id: 1, name: "John" });
    }, 1000);
  });
}

// ===== Async/Await with Types =====
interface User {
  id: number;
  name: string;
  email: string;
}

async function getUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const user: User = await response.json();
  return user;
}

// Usage
const user = await getUser(1); // Type: User

// ===== Error Handling with Types =====
interface ApiError {
  message: string;
  code: number;
}

type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: ApiError };

async function fetchData<T>(url: string): Promise<Result<T>> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      return {
        success: false,
        error: { message: "Request failed", code: response.status }
      };
    }
    const data = await response.json();
    return { success: true, data };
  } catch (error) {
    return {
      success: false,
      error: { message: "Network error", code: 0 }
    };
  }
}

// Usage with type narrowing
const result = await fetchData<User>("/api/user/1");
if (result.success) {
  console.log(result.data.name); // ✅ TypeScript knows data exists
} else {
  console.error(result.error.message); // ✅ TypeScript knows error exists
}

// ===== Promise.all with Types =====
async function loadMultiple() {
  const [users, products, orders] = await Promise.all([
    fetch("/api/users").then(r => r.json()) as Promise<User[]>,
    fetch("/api/products").then(r => r.json()) as Promise<Product[]>,
    fetch("/api/orders").then(r => r.json()) as Promise<Order[]>
  ]);

  // All properly typed
  users.forEach(u => console.log(u.name));
  products.forEach(p => console.log(p.price));
  orders.forEach(o => console.log(o.total));
}

// ===== Generic Async Function =====
async function fetchResource<T>(
  endpoint: string,
  validator: (data: unknown) => data is T
): Promise<T> {
  const response = await fetch(endpoint);
  const data = await response.json();

  if (!validator(data)) {
    throw new Error("Invalid data format");
  }

  return data;
}

// Type guard
function isUser(data: unknown): data is User {
  return (
    typeof data === "object" &&
    data !== null &&
    "id" in data &&
    "name" in data &&
    "email" in data
  );
}

// Usage
const user = await fetchResource("/api/user/1", isUser);
console.log(user.email); // ✅ Properly typed

// ===== Retry Logic with Types =====
async function retry<T>(
  fn: () => Promise<T>,
  maxAttempts: number = 3
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${attempt} failed`);
    }
  }

  throw lastError!;
}

// Usage
const data = await retry(() => fetchUser(), 3);
```

**Think of it as:** 
- 🎯 **Type safety for async operations** - Promises and async/await with proper types
- Prevents runtime errors by catching type issues at compile time

---

### ⭐ tsconfig.json Key Options
**Simple Explanation:** The `tsconfig.json` file configures how TypeScript compiles your code. Key options control strictness, output, and module resolution.

```json
{
  "compilerOptions": {
    /* Language & Environment */
    "target": "ES2020",                    // JavaScript version to compile to
    "lib": ["ES2020", "DOM"],              // Include type definitions
    "experimentalDecorators": true,        // Enable decorators
    "emitDecoratorMetadata": true,         // Metadata for decorators

    /* Modules */
    "module": "commonjs",                  // Module system (commonjs, ES2015, ESNext)
    "moduleResolution": "node",            // How modules are resolved
    "baseUrl": "./",                       // Base directory for non-relative imports
    "paths": {                             // Path mapping (like aliases)
      "@models/*": ["src/models/*"],
      "@utils/*": ["src/utils/*"]
    },
    "rootDir": "./src",                    // Root folder of source files
    "outDir": "./dist",                    // Output folder for compiled files

    /* Type Checking - STRICT MODE */
    "strict": true,                        // Enable all strict checks
    "noImplicitAny": true,                 // Error on 'any' type
    "strictNullChecks": true,              // null/undefined must be explicit
    "strictFunctionTypes": true,           // Strict function type checking
    "strictBindCallApply": true,           // Strict bind/call/apply
    "strictPropertyInitialization": true,  // Class properties must be initialized
    "noImplicitThis": true,                // Error on 'this' with implicit 'any'
    "alwaysStrict": true,                  // Use 'use strict' in output

    /* Additional Checks */
    "noUnusedLocals": true,                // Error on unused local variables
    "noUnusedParameters": true,            // Error on unused parameters
    "noImplicitReturns": true,             // Error if function doesn't return in all paths
    "noFallthroughCasesInSwitch": true,    // Error on fallthrough in switch

    /* Emit */
    "declaration": true,                   // Generate .d.ts files
    "declarationMap": true,                // Source maps for .d.ts files
    "sourceMap": true,                     // Generate .map files for debugging
    "removeComments": true,                // Remove comments from output
    "noEmit": false,                       // Don't emit output (useful for type-checking only)
    "importHelpers": true,                 // Import helpers from tslib (smaller output)

    /* Interop Constraints */
    "esModuleInterop": true,               // Better interop with CommonJS
    "allowSyntheticDefaultImports": true,  // Allow default imports from modules with no default export
    "forceConsistentCasingInFileNames": true, // Enforce consistent casing

    /* Skip Lib Check */
    "skipLibCheck": true                   // Skip type checking of .d.ts files (faster compilation)
  },

  /* Include/Exclude */
  "include": ["src/**/*"],                 // Files to include
  "exclude": ["node_modules", "dist"]      // Files to exclude
}
```

**Common Configurations:**

```json
// React Project
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["src"]
}

// Node.js Project
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
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}

// Library Project
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "ESNext",
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

**Think of it as:** 
- ⚙️ **Control panel for TypeScript** - Configure how strict and what output you want
- **Enable strict mode** for maximum type safety

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

## 21. Enums
**Concept:** Enums (Enumerations) define a set of named constants, making code more readable and maintainable than using raw numbers or strings.

### Types of Enums:

```typescript
// 1. Numeric Enums (Default)
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right    // 3
}

let dir: Direction = Direction.Up;
console.log(dir); // 0

// Custom starting value
enum Status {
  Pending = 1,
  Approved,    // 2
  Rejected     // 3
}

// 2. String Enums (Recommended for APIs)
enum OrderStatus {
  Pending = "PENDING",
  Processing = "PROCESSING",
  Shipped = "SHIPPED",
  Delivered = "DELIVERED"
}

let status: OrderStatus = OrderStatus.Shipped;
console.log(status); // "SHIPPED"

// 3. Const Enums (Optimized - inlined at compile time)
const enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE"
}

let color = Color.Red; // Compiled as "RED" (no runtime enum object)

// 4. Computed Enums
enum FileAccess {
  None = 0,
  Read = 1 << 1,           // 2
  Write = 1 << 2,          // 4
  ReadWrite = Read | Write // 6
}

// Reverse mapping (Numeric enums only)
enum Animal {
  Dog = 1,
  Cat = 2
}

console.log(Animal[1]); // "Dog"
console.log(Animal.Dog); // 1
```

**Use Cases:**
- HTTP status codes
- API response states
- User roles/permissions
- Configuration options
- State machine states

**Best Practice:** Use string enums for external APIs (better debugging), numeric enums for internal flags.

---

## 22. Intersection Types
**Concept:** Intersection types combine multiple types into one that has all properties from each type using the `&` operator.

**Example:**
```typescript
// Basic intersection
interface Identifiable {
  id: number;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

interface Nameable {
  name: string;
}

// Combine all three
type Entity = Identifiable & Timestamped & Nameable;

const user: Entity = {
  id: 1,
  name: "John",
  createdAt: new Date(),
  updatedAt: new Date()
};

// Practical example - Mixing capabilities
type Draggable = {
  drag(): void;
};

type Resizable = {
  resize(): void;
};

type Rotatable = {
  rotate(angle: number): void;
};

type UIWidget = Draggable & Resizable & Rotatable;

const widget: UIWidget = {
  drag() { console.log("Dragging"); },
  resize() { console.log("Resizing"); },
  rotate(angle) { console.log(`Rotating ${angle}°`); }
};

// Intersecting with primitives (results in never)
type Impossible = string & number; // never (can't be both)

// Useful pattern - Extending base types
type BaseUser = {
  id: number;
  email: string;
};

type AdminUser = BaseUser & {
  permissions: string[];
  role: "admin";
};

type GuestUser = BaseUser & {
  role: "guest";
  expiresAt: Date;
};
```

**Difference:**
- **Union (`|`)**: Value can be ONE of the types (A or B)
- **Intersection (`&`)**: Value must satisfy ALL types (A and B)

---

## 23. Discriminated Unions (Tagged Unions)
**Concept:** A pattern using a common literal property (discriminator) to distinguish between types in a union, enabling exhaustive type checking.

**Example:**
```typescript
// Payment methods with discriminator
interface CreditCardPayment {
  kind: "credit"; // discriminator
  cardNumber: string;
  cvv: string;
}

interface PayPalPayment {
  kind: "paypal";
  email: string;
}

interface BankTransfer {
  kind: "bank";
  accountNumber: string;
  routingNumber: string;
}

type Payment = CreditCardPayment | PayPalPayment | BankTransfer;

function processPayment(payment: Payment) {
  switch (payment.kind) {
    case "credit":
      // TypeScript knows payment is CreditCardPayment
      console.log(`Processing card: ${payment.cardNumber}`);
      break;
    case "paypal":
      // TypeScript knows payment is PayPalPayment
      console.log(`Processing PayPal: ${payment.email}`);
      break;
    case "bank":
      // TypeScript knows payment is BankTransfer
      console.log(`Processing bank: ${payment.accountNumber}`);
      break;
    default:
      // Exhaustiveness check
      const _exhaustive: never = payment;
      throw new Error(`Unhandled payment type: ${_exhaustive}`);
  }
}

// API Response example
interface SuccessResponse<T> {
  status: "success";
  data: T;
}

interface ErrorResponse {
  status: "error";
  message: string;
  code: number;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function handleResponse<T>(response: ApiResponse<T>): T {
  if (response.status === "success") {
    return response.data; // TypeScript knows data exists
  } else {
    throw new Error(`Error ${response.code}: ${response.message}`);
  }
}
```

**Benefits:**
- Type-safe exhaustive checking
- Compiler ensures all cases are handled
- Auto-completion for each variant
- Refactoring safety

---

## 24. `keyof` Operator
**Concept:** The `keyof` operator creates a union type of all property keys of a given type.

**Example:**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// keyof creates union of keys
type UserKeys = keyof User; // "id" | "name" | "email" | "age"

// Generic property getter
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
  age: 30
};

const name = getProperty(user, "name");   // Type: string
const age = getProperty(user, "age");     // Type: number
// const invalid = getProperty(user, "salary"); // Error: not a key

// Dynamic property updates
function updateProperty<T, K extends keyof T>(
  obj: T,
  key: K,
  value: T[K]
): void {
  obj[key] = value;
}

updateProperty(user, "age", 31);           // ✅ OK
// updateProperty(user, "age", "thirty");  // ❌ Error: must be number

// Create pick function
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  keys.forEach(key => {
    result[key] = obj[key];
  });
  return result;
}

const subset = pick(user, ["name", "email"]);
// Type: { name: string; email: string }
```

---

## 25. `typeof` Operator
**Concept:** The `typeof` operator in TypeScript's type system extracts the type from a value, enabling type inference from existing objects or functions.

**Example:**
```typescript
// Extract type from object
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retryAttempts: 3
};

type Config = typeof config;
// { apiUrl: string; timeout: number; retryAttempts: number }

function createConfig(overrides: Partial<Config>): Config {
  return { ...config, ...overrides };
}

// Extract function signature
function greet(name: string, age: number): string {
  return `Hello ${name}, you are ${age}`;
}

type GreetFunction = typeof greet;
// (name: string, age: number) => string

type GreetParams = Parameters<typeof greet>;
// [string, number]

type GreetReturn = ReturnType<typeof greet>;
// string

// With const assertions
const routes = {
  home: "/",
  about: "/about",
  contact: "/contact"
} as const;

type Routes = typeof routes;
// {
//   readonly home: "/";
//   readonly about: "/about";
//   readonly contact: "/contact";
// }

type RouteKeys = keyof typeof routes; // "home" | "about" | "contact"

// Extract enum type
enum UserRole {
  Admin = "ADMIN",
  User = "USER",
  Guest = "GUEST"
}

type RoleEnum = typeof UserRole;
// {
//   Admin: UserRole.Admin;
//   User: UserRole.User;
//   Guest: UserRole.Guest;
// }

// Practical example - Infer from implementation
const validators = {
  email: (value: string) => /\S+@\S+\.\S+/.test(value),
  minLength: (value: string, min: number) => value.length >= min,
  isNumber: (value: string) => !isNaN(Number(value))
};

type Validators = typeof validators;
type EmailValidator = typeof validators.email;
// (value: string) => boolean
```

**Use Cases:**
- Inferring types from configuration objects
- Creating types from existing values
- Working with third-party libraries without type definitions

---

## 26. Indexed Access Types
**Concept:** Access the type of a property within another type using bracket notation.

**Example:**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  profile: {
    bio: string;
    avatar: string;
    socialLinks: {
      twitter?: string;
      linkedin?: string;
    };
  };
}

// Access specific property types
type UserId = User["id"];           // number
type UserName = User["name"];       // string
type UserProfile = User["profile"]; // { bio: string; avatar: string; ... }

// Deep property access
type Bio = User["profile"]["bio"];  // string
type SocialLinks = User["profile"]["socialLinks"]; 
// { twitter?: string; linkedin?: string }

// Multiple property access (creates union)
type UserContactInfo = User["name" | "email"]; // string (both are strings)
type IdOrName = User["id" | "name"];           // string | number

// Array element type
type UserArray = User[];
type ArrayElement = UserArray[number]; // User

// Generic indexed access
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Practical example - Extract nested types
interface ApiResponse {
  data: {
    users: Array<{
      id: number;
      profile: {
        firstName: string;
        lastName: string;
        address: {
          city: string;
          country: string;
        };
      };
    }>;
  };
}

type UserData = ApiResponse["data"]["users"][number];
// { id: number; profile: { ... } }

type UserProfile = UserData["profile"];
// { firstName: string; lastName: string; address: { ... } }

type Address = UserProfile["address"];
// { city: string; country: string }

// With utility types
type ReadonlyUser = Readonly<User>;
type ReadonlyEmail = ReadonlyUser["email"]; // string (readonly modifier doesn't affect type)

// Conditional indexed access
type NonNullableKeys<T> = {
  [K in keyof T]: null extends T[K] ? never : K;
}[keyof T];

interface MaybeUser {
  id: number;
  name: string | null;
  email: string;
}

type RequiredKeys = NonNullableKeys<MaybeUser>; // "id" | "email"
```

---

## 27. Mapped Types
**Concept:** Transform all properties of an existing type using a mapping operation. Like `Array.map()` but for types.

**Example:**
```typescript
// Built-in mapped types reimplemented
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

type MyRequired<T> = {
  [K in keyof T]-?: T[K]; // -? removes optionality
};

// Remove readonly modifier
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

// Make all properties nullable
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

interface User {
  id: number;
  name: string;
  email: string;
}

type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; email: string | null }

// Key remapping with template literals
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// {
//   getId: () => number;
//   getName: () => string;
//   getEmail: () => string;
// }

// Setters
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

// Filtering properties
type StringProperties<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K];
};

interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
}

type ProductStrings = StringProperties<Product>;
// { name: string; description: string }

// Exclude specific keys
type OmitId<T> = {
  [K in keyof T as K extends "id" ? never : K]: T[K];
};

type ProductWithoutId = OmitId<Product>;
// { name: string; description: string; price: number }

// Conditional mapped types
type AsyncMethods<T> = {
  [K in keyof T]: T[K] extends (...args: any[]) => any
    ? (...args: Parameters<T[K]>) => Promise<ReturnType<T[K]>>
    : T[K];
};

interface SyncService {
  getData(): string;
  count: number;
  process(value: number): boolean;
}

type AsyncService = AsyncMethods<SyncService>;
// {
//   getData: () => Promise<string>;
//   count: number;
//   process: (value: number) => Promise<boolean>;
// }
```

---

## 28. Conditional Types
**Concept:** Types that select one of two possible types based on a condition, expressed as `T extends U ? X : Y`.

**Example:**
```typescript
// Basic conditional type
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>;  // "yes"
type B = IsString<number>;  // "no"

// Extract function return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>; // { id: number; name: string }

// Extract array element type
type ElementType<T> = T extends (infer U)[] ? U : T;

type StringElement = ElementType<string[]>; // string
type NumberType = ElementType<number>;      // number

// Unwrap Promise
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;

type Result1 = Awaited<Promise<string>>;           // string
type Result2 = Awaited<Promise<Promise<number>>>;  // number

// Exclude/Extract implementations
type MyExclude<T, U> = T extends U ? never : T;
type MyExtract<T, U> = T extends U ? T : never;

type Colors = "red" | "green" | "blue" | "yellow";
type PrimaryColors = MyExtract<Colors, "red" | "blue">; // "red" | "blue"
type NonPrimary = MyExclude<Colors, "red" | "blue">;    // "green" | "yellow"

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>; // string[] | number[]

// Non-distributive (wrap in tuple)
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type Result2 = ToArrayNonDist<string | number>; // (string | number)[]

// Practical example - Function parameter type
type FirstParameter<T> = T extends (first: infer F, ...args: any[]) => any
  ? F
  : never;

function process(id: number, name: string, active: boolean) {}

type FirstParam = FirstParameter<typeof process>; // number

// Nested conditional types
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";

type T0 = TypeName<string>;    // "string"
type T1 = TypeName<number>;    // "number"
type T2 = TypeName<() => void>; // "function"

// Flatten nested arrays
type DeepFlatten<T> = T extends Array<infer U>
  ? DeepFlatten<U>
  : T;

type Nested = string[][][];
type Flat = DeepFlatten<Nested>; // string
```

---

## 29. Template Literal Types
**Concept:** Construct new string literal types by combining existing string literal types using template literal syntax.

**Example:**
```typescript
// Basic template literal
type World = "world";
type Greeting = `hello ${World}`; // "hello world"

// Combining multiple literals
type Color = "red" | "green" | "blue";
type Size = "small" | "medium" | "large";
type ColoredSize = `${Color}-${Size}`;
// "red-small" | "red-medium" | "red-large" | "green-small" | ...

// Event handlers
type EventName = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// CRUD operations
type Entity = "user" | "product" | "order";
type Action = "create" | "read" | "update" | "delete";
type Permission = `${Action}:${Entity}`;
// "create:user" | "read:user" | "update:user" | ...

// HTTP endpoints
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Route = "/users" | "/products" | "/orders";
type Endpoint = `${HttpMethod} ${Route}`;
// "GET /users" | "POST /users" | ...

// CSS class names
type Breakpoint = "sm" | "md" | "lg" | "xl";
type Property = "margin" | "padding";
type Direction = "top" | "right" | "bottom" | "left";
type UtilityClass = `${Breakpoint}:${Property}-${Direction}`;
// "sm:margin-top" | "sm:margin-right" | ...

// Intrinsic string manipulation types
type UppercaseGreeting = Uppercase<"hello">; // "HELLO"
type LowercaseGreeting = Lowercase<"HELLO">; // "hello"
type CapitalizedName = Capitalize<"john">;   // "John"
type UncapitalizedName = Uncapitalize<"John">; // "john"

// Practical example - Type-safe event emitter
type EventMap = {
  userLoggedIn: { userId: number; timestamp: Date };
  userLoggedOut: { userId: number };
  orderPlaced: { orderId: string; total: number };
};

type EventKey = keyof EventMap;
type EventCallback<K extends EventKey> = (data: EventMap[K]) => void;

class TypedEventEmitter {
  on<K extends EventKey>(event: K, callback: EventCallback<K>): void {
    // Implementation
  }

  emit<K extends EventKey>(event: K, data: EventMap[K]): void {
    // Implementation
  }
}

const emitter = new TypedEventEmitter();
emitter.on("userLoggedIn", (data) => {
  console.log(data.userId, data.timestamp); // Fully typed!
});

// API versioning
type ApiVersion = "v1" | "v2" | "v3";
type Resource = "users" | "posts" | "comments";
type ApiEndpoint = `/api/${ApiVersion}/${Resource}`;
// "/api/v1/users" | "/api/v1/posts" | ...

// Database table names
type TableName<T extends string> = `tbl_${Lowercase<T>}`;
type UserTable = TableName<"User">; // "tbl_user"

// Getters and Setters
type GetterName<T extends string> = `get${Capitalize<T>}`;
type SetterName<T extends string> = `set${Capitalize<T>}`;

type UserGetter = GetterName<"name">; // "getName"
type UserSetter = SetterName<"name">; // "setName"
```

---

## 30. Function Overloads
**Concept:** Define multiple signatures for a single function, allowing different parameter and return types based on input.

**Example:**
```typescript
// Basic overload
function format(value: string): string;
function format(value: number): string;
function format(value: boolean): string;
function format(value: string | number | boolean): string {
  return String(value);
}

const s = format("hello");    // Type: string
const n = format(42);         // Type: string
const b = format(true);       // Type: string

// Different return types based on input
function getRecord(id: number): { id: number; name: string };
function getRecord(email: string): { id: number; email: string };
function getRecord(param: number | string): any {
  if (typeof param === "number") {
    return { id: param, name: "John" };
  } else {
    return { id: 1, email: param };
  }
}

const record1 = getRecord(1);          // Type: { id: number; name: string }
const record2 = getRecord("j@j.com");  // Type: { id: number; email: string }

// Array-like access
interface User {
  id: number;
  name: string;
  email: string;
}

function getUserInfo(id: number): User;
function getUserInfo(ids: number[]): User[];
function getUserInfo(param: number | number[]): User | User[] {
  if (typeof param === "number") {
    return { id: param, name: "John", email: "j@j.com" };
  } else {
    return param.map(id => ({ id, name: "User", email: "user@example.com" }));
  }
}

const singleUser = getUserInfo(1);      // Type: User
const multipleUsers = getUserInfo([1, 2]); // Type: User[]

// Optional parameters in overloads
function createElement(tag: "div"): HTMLDivElement;
function createElement(tag: "span"): HTMLSpanElement;
function createElement(tag: "a", href: string): HTMLAnchorElement;
function createElement(tag: string, href?: string): HTMLElement {
  const element = document.createElement(tag);
  if (tag === "a" && href) {
    (element as HTMLAnchorElement).href = href;
  }
  return element;
}

const div = createElement("div");           // Type: HTMLDivElement
const link = createElement("a", "#");       // Type: HTMLAnchorElement

// Method overloads in classes
class DataStore {
  add(item: string): void;
  add(items: string[]): void;
  add(param: string | string[]): void {
    if (typeof param === "string") {
      // Add single item
    } else {
      // Add multiple items
    }
  }

  get(index: number): string;
  get(range: [number, number]): string[];
  get(param: number | [number, number]): string | string[] {
    if (typeof param === "number") {
      return "item";
    } else {
      return ["item1", "item2"];
    }
  }
}

// Generic overloads
function map<T, U>(array: T[], fn: (item: T) => U): U[];
function map<T, U>(array: T[], fn: (item: T, index: number) => U): U[];
function map<T, U>(
  array: T[],
  fn: (item: T, index?: number) => U
): U[] {
  return array.map(fn);
}
```

**Best Practices:**
- Order overloads from most specific to least specific
- Implementation signature must be compatible with all overloads
- Don't overuse – consider union types or generics instead

---

## 31. Decorators
**Concept:** Special declarations that can be attached to classes, methods, properties, or parameters to modify their behavior. Widely used in frameworks like Angular and NestJS.

**Enable in tsconfig.json:**
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

**Example:**
```typescript
// 1. Class Decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

// 2. Method Decorator with logging
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// Output:
// Calling add with: [2, 3]
// Result: 5

// 3. Property Decorator
function readonly(target: any, propertyKey: string) {
  Object.defineProperty(target, propertyKey, {
    writable: false
  });
}

class Person {
  @readonly
  name: string = "John";
}

const person = new Person();
// person.name = "Jane"; // Error in strict mode

// 4. Parameter Decorator
function validate(
  target: any,
  propertyKey: string,
  parameterIndex: number
) {
  console.log(`Parameter ${parameterIndex} in ${propertyKey} needs validation`);
}

class UserService {
  getUser(@validate id: number): void {
    console.log(`Getting user ${id}`);
  }
}

// 5. Decorator Factory (with parameters)
function MinLength(min: number) {
  return function (target: any, propertyKey: string) {
    let value: string;

    const getter = () => value;
    const setter = (newVal: string) => {
      if (newVal.length < min) {
        throw new Error(`${propertyKey} must be at least ${min} characters`);
      }
      value = newVal;
    };

    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter
    });
  };
}

class User {
  @MinLength(3)
  username: string;

  constructor(username: string) {
    this.username = username;
  }
}

// 6. Authorization decorator
function Authorize(roles: string[]) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const userRole = "admin"; // Would come from auth context
      if (!roles.includes(userRole)) {
        throw new Error("Unauthorized");
      }
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class AdminController {
  @Authorize(["admin"])
  deleteUser(userId: number): void {
    console.log(`Deleting user ${userId}`);
  }

  @Authorize(["admin", "moderator"])
  banUser(userId: number): void {
    console.log(`Banning user ${userId}`);
  }
}

// 7. Timing decorator
function Timing(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = async function (...args: any[]) {
    const start = performance.now();
    const result = await originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} took ${end - start}ms`);
    return result;
  };

  return descriptor;
}

class DataService {
  @Timing
  async fetchData(): Promise<any> {
    await new Promise(resolve => setTimeout(resolve, 1000));
    return { data: "example" };
  }
}

// 8. Multiple decorators (execute bottom to top)
function First() {
  console.log("First(): factory");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("First(): called");
  };
}

function Second() {
  console.log("Second(): factory");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("Second(): called");
  };
}

class Example {
  @First()
  @Second()
  method() {}
}
// Output:
// First(): factory
// Second(): factory
// Second(): called
// First(): called
```

---

## 32. Modules & Namespaces
**Concept:** Organize code into reusable, maintainable units. **ES6 Modules** are the modern standard; **Namespaces** are TypeScript's older internal module system.

**Example:**
```typescript
// ===== ES6 Modules (Recommended) =====

// --- math.ts ---
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export const PI = 3.14159;

export default class Calculator {
  multiply(a: number, b: number): number {
    return a * b;
  }
}

// --- app.ts ---
// Named imports
import { add, subtract, PI } from "./math";

// Default import
import Calculator from "./math";

// Alias imports
import { add as sum } from "./math";

// Import all
import * as MathUtils from "./math";

// Re-export
export { add, subtract } from "./math";
export { default as Calc } from "./math";

// ===== Namespaces (Legacy) =====

// --- utilities.ts ---
namespace StringUtils {
  export function capitalize(str: string): string {
    return str.charAt(0).toUpperCase() + str.slice(1);
  }

  export function reverse(str: string): string {
    return str.split("").reverse().join("");
  }

  // Not exported - internal to namespace
  function helper() {
    return "internal";
  }
}

// Usage
const result = StringUtils.capitalize("hello");

// Nested namespaces
namespace App {
  export namespace Models {
    export interface User {
      id: number;
      name: string;
    }
  }

  export namespace Services {
    export class UserService {
      getUser(id: number): Models.User {
        return { id, name: "John" };
      }
    }
  }
}

const userService = new App.Services.UserService();

// ===== Module Augmentation =====

// Extend existing module
declare module "express" {
  interface Request {
    user?: {
      id: number;
      name: string;
    };
  }
}

// Now Request has user property
import { Request } from "express";

function handler(req: Request) {
  console.log(req.user?.id); // ✅ user property exists
}

// Global augmentation
declare global {
  interface Window {
    myCustomProperty: string;
  }
}

window.myCustomProperty = "value"; // ✅ OK

// ===== Dynamic imports =====
async function loadModule() {
  const module = await import("./math");
  const result = module.add(2, 3);
}

// ===== Barrel exports (index.ts pattern) =====

// --- models/index.ts ---
export * from "./user";
export * from "./product";
export * from "./order";

// --- app.ts ---
import { User, Product, Order } from "./models";
```

**Best Practices:**
- Use ES6 modules for all new code
- Prefer named exports over default exports
- Use barrel exports for cleaner imports
- Avoid namespaces unless maintaining legacy code

---

## 33. Async/Await & Promise Types
**Concept:** Type-safe handling of asynchronous operations using Promises and async/await.

**Example:**
```typescript
// Basic Promise types
function fetchUser(id: number): Promise<User> {
  return fetch(`/api/users/${id}`)
    .then(response => response.json())
    .then(data => data as User);
}

// Async function automatically returns Promise
async function getUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const user: User = await response.json();
  return user; // Wrapped in Promise automatically
}

// Generic async function
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return await response.json();
}

const user = await fetchData<User>("/api/user/1");

// Error handling with types
interface ApiError {
  message: string;
  code: number;
}

type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: ApiError };

async function safeRequest<T>(url: string): Promise<Result<T>> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      return {
        success: false,
        error: {
          message: response.statusText,
          code: response.status
        }
      };
    }
    const data = await response.json();
    return { success: true, data };
  } catch (error) {
    return {
      success: false,
      error: {
        message: error instanceof Error ? error.message : "Unknown error",
        code: 0
      }
    };
  }
}

// Usage with type narrowing
const result = await safeRequest<User>("/api/user/1");
if (result.success) {
  console.log(result.data.name); // ✅ TypeScript knows data exists
} else {
  console.error(result.error.message); // ✅ TypeScript knows error exists
}

// Promise.all with types
async function loadAllData() {
  const [users, products, orders] = await Promise.all([
    fetchData<User[]>("/api/users"),
    fetchData<Product[]>("/api/products"),
    fetchData<Order[]>("/api/orders")
  ]);
  
  // All properly typed
  users.forEach(u => console.log(u.name));
  products.forEach(p => console.log(p.price));
  orders.forEach(o => console.log(o.total));
}

// Promise.race with types
async function raceRequests(): Promise<User> {
  const promises = [
    fetchData<User>("/api/user/fast"),
    fetchData<User>("/api/user/slow")
  ];
  
  return await Promise.race(promises);
}

// Awaited utility type (TypeScript 4.5+)
type PromiseValue = Awaited<Promise<string>>; // string
type NestedPromise = Awaited<Promise<Promise<number>>>; // number

// Retry logic with types
async function retry<T>(
  fn: () => Promise<T>,
  maxAttempts: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      if (attempt < maxAttempts) {
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError!;
}

// Type-safe async iterator
async function* generateNumbers(count: number): AsyncGenerator<number> {
  for (let i = 0; i < count; i++) {
    await new Promise(resolve => setTimeout(resolve, 100));
    yield i;
  }
}

for await (const num of generateNumbers(5)) {
  console.log(num); // 0, 1, 2, 3, 4
}

// Practical example - Paginated API
interface PaginatedResponse<T> {
  data: T[];
  page: number;
  totalPages: number;
}

async function fetchAllPages<T>(
  baseUrl: string
): Promise<T[]> {
  const allData: T[] = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const response = await fetchData<PaginatedResponse<T>>(
      `${baseUrl}?page=${page}`
    );
    allData.push(...response.data);
    hasMore = page < response.totalPages;
    page++;
  }

  return allData;
}

const allUsers = await fetchAllPages<User>("/api/users");
```

---

## 34. tsconfig.json Configuration
**Concept:** The `tsconfig.json` file configures the TypeScript compiler, controlling strictness, module resolution, and output.

**Example:**
```json
{
  "compilerOptions": {
    /* Language & Environment */
    "target": "ES2020",                      // Output JavaScript version
    "lib": ["ES2020", "DOM"],                // Include type definitions
    "jsx": "react-jsx",                      // JSX support for React
    "experimentalDecorators": true,          // Enable decorators
    "emitDecoratorMetadata": true,           // Metadata for decorators
    "useDefineForClassFields": true,         // Modern class field behavior

    /* Modules */
    "module": "ESNext",                      // Module system
    "moduleResolution": "bundler",           // How to resolve modules
    "baseUrl": "./",                         // Base for non-relative imports
    "paths": {                               // Path mapping (aliases)
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    },
    "rootDir": "./src",                      // Root of source files
    "outDir": "./dist",                      // Output directory
    "resolveJsonModule": true,               // Import JSON files
    "allowImportingTsExtensions": false,     // .ts in imports

    /* Type Checking - STRICT MODE */
    "strict": true,                          // Enable ALL strict checks
    "noImplicitAny": true,                   // Error on implicit 'any'
    "strictNullChecks": true,                // null/undefined explicit
    "strictFunctionTypes": true,             // Strict function checking
    "strictBindCallApply": true,             // Strict bind/call/apply
    "strictPropertyInitialization": true,    // Class properties initialized
    "noImplicitThis": true,                  // No implicit 'any' this
    "useUnknownInCatchVariables": true,      // catch variables as unknown
    "alwaysStrict": true,                    // Use 'use strict'

    /* Additional Checks */
    "noUnusedLocals": true,                  // Error on unused variables
    "noUnusedParameters": true,              // Error on unused parameters
    "noImplicitReturns": true,               // All paths must return
    "noFallthroughCasesInSwitch": true,      // No switch fallthrough
    "noUncheckedIndexedAccess": true,        // Index access returns T | undefined
    "noImplicitOverride": true,              // Explicit override keyword
    "allowUnusedLabels": false,              // Error on unused labels
    "allowUnreachableCode": false,           // Error on unreachable code

    /* Emit */
    "declaration": true,                     // Generate .d.ts files
    "declarationMap": true,                  // Source maps for .d.ts
    "sourceMap": true,                       // Generate .map files
    "inlineSourceMap": false,                // Inline source maps
    "removeComments": true,                  // Remove comments in output
    "noEmit": false,                         // Don't emit (type-check only)
    "importHelpers": true,                   // Import helpers from tslib
    "downlevelIteration": true,              // Full iterator support

    /* Interop Constraints */
    "esModuleInterop": true,                 // Better CommonJS interop
    "allowSyntheticDefaultImports": true,    // Allow default imports
    "forceConsistentCasingInFileNames": true, // Consistent file casing
    "isolatedModules": true,                 // Each file as separate module

    /* Skip Lib Check */
    "skipLibCheck": true                     // Skip .d.ts type checking
  },

  /* Include/Exclude */
  "include": ["src/**/*"],                   // Files to include
  "exclude": [                               // Files to exclude
    "node_modules",
    "dist",
    "**/*.spec.ts"
  ]
}
```

**Common Configurations:**

```json
// React/Vite Project
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}

// Node.js/Express Project
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
    "sourceMap": true,
    "declaration": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}

// Library/Package
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "ESNext",
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "node"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
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
| Enums | Named constants for better readability (numeric, string, const) |
| Intersection Types | Combine multiple types with `&` - must have all properties |
| Discriminated Unions | Use common property to distinguish types in unions |
| `keyof` | Extract property keys as union type |
| `typeof` | Extract type from value |
| Indexed Access | Access property types using bracket notation |
| Mapped Types | Transform properties of existing types |
| Conditional Types | Choose types based on conditions (`T extends U ? X : Y`) |
| Template Literals | Build string literal types with template syntax |
| Function Overloads | Multiple signatures for different parameter/return types |
| Decorators | Modify classes/methods with `@` syntax (Angular, NestJS) |
| Modules/Namespaces | ES6 modules (modern) vs namespaces (legacy) |
| Async/Await Types | Type-safe Promises and async operations |
| tsconfig.json | Configure compiler options, strict mode, modules |
