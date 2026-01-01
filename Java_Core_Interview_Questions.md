# Java Core Interview Questions

## 1. Java Functional Interface
**Concept:** A functional interface is an interface with **exactly one abstract method** (SAM - Single Abstract Method). It can have multiple default or static methods, but only one unimplemented method.

**Why do we need it?**
Functional interfaces enable **Lambda Expressions** and **Method References**, making code more concise and readable.

### Built-in Functional Interfaces (java.util.function package):

| Interface | Method | Input | Output | Use Case |
|-----------|--------|-------|--------|----------|
| `Predicate<T>` | `test(T t)` | T | boolean | Filtering/Validation |
| `Function<T, R>` | `apply(T t)` | T | R | Transformation |
| `Consumer<T>` | `accept(T t)` | T | void | Processing/Side effects |
| `Supplier<T>` | `get()` | none | T | Generation/Factory |
| `BiFunction<T, U, R>` | `apply(T t, U u)` | T, U | R | Two inputs to one output |

**Example:**
```java
// Custom Functional Interface
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // Can have default methods
    default int square(int x) {
        return x * x;
    }
}

// Usage with Lambda
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.calculate(5, 3));      // 8
System.out.println(multiply.calculate(5, 3));  // 15

// Built-in Functional Interfaces
// 1. Predicate - Filtering
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
Predicate<Integer> isEven = num -> num % 2 == 0;
numbers.stream()
    .filter(isEven)
    .forEach(System.out::println); // 2, 4, 6

// 2. Function - Transformation
Function<String, Integer> stringLength = str -> str.length();
System.out.println(stringLength.apply("Hello")); // 5

// 3. Consumer - Processing
Consumer<String> print = str -> System.out.println(str);
print.accept("Hello World"); // Hello World

// 4. Supplier - Generation
Supplier<Double> randomValue = () -> Math.random();
System.out.println(randomValue.get()); // Random number

// 5. BiFunction - Two inputs
BiFunction<Integer, Integer, Integer> power = (base, exp) -> (int) Math.pow(base, exp);
System.out.println(power.apply(2, 3)); // 8
```

---

## 2. Abstraction in Java
**Concept:** Abstraction means **hiding implementation details** and showing only the essential features. It's achieved using **Abstract Classes** and **Interfaces**.

### Abstract Class:
- Can have abstract methods (without body) and concrete methods (with body)
- Can have constructors
- Can have instance variables
- A class can extend only **one** abstract class

**Example:**
```java
// Abstract class
abstract class Animal {
    protected String name;
    
    // Constructor
    public Animal(String name) {
        this.name = name;
    }
    
    // Abstract method - no implementation
    abstract void makeSound();
    
    // Concrete method - has implementation
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

// Concrete class
class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
    
    @Override
    void makeSound() {
        System.out.println("Woof!");
    }
}

// Usage
Dog dog = new Dog("Buddy");
dog.makeSound(); // Woof!
dog.sleep();     // Buddy is sleeping
```

### Interface:
- All methods are abstract by default (before Java 8)
- From Java 8: Can have default and static methods
- From Java 9: Can have private methods
- Cannot have constructors
- All variables are `public static final` by default
- A class can implement **multiple** interfaces

**Example:**
```java
interface Flyable {
    void fly(); // abstract by default
    
    // Default method (Java 8+)
    default void takeOff() {
        System.out.println("Taking off...");
    }
}

interface Swimmable {
    void swim();
}

// A class can implement multiple interfaces
class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("Duck is flying");
    }
    
    @Override
    public void swim() {
        System.out.println("Duck is swimming");
    }
}

// Usage
Duck duck = new Duck();
duck.takeOff(); // Taking off...
duck.fly();     // Duck is flying
duck.swim();    // Duck is swimming
```

### When to use what?

| Use Case | Abstract Class | Interface |
|----------|---------------|-----------|
| Multiple inheritance | ❌ No | ✅ Yes |
| Has state (instance variables) | ✅ Yes | ❌ No (only constants) |
| Has constructors | ✅ Yes | ❌ No |
| Has implementation | ✅ Can have concrete methods | ⚠️ Only default/static (Java 8+) |
| Relationship | "IS-A" (inheritance) | "CAN-DO" (capability) |

---

## 3. OOP Concepts in Java

### 1. Encapsulation
**Concept:** Bundling data (variables) and methods that operate on the data into a single unit (class), and restricting direct access to some components.

```java
public class BankAccount {
    // Private variables (data hiding)
    private String accountNumber;
    private double balance;
    
    // Constructor
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }
    
    // Public getters and setters (controlled access)
    public double getBalance() {
        return balance;
    }
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public boolean withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }
}
```

### 2. Inheritance
**Concept:** A class can inherit properties and methods from another class.

```java
// Parent class
class Vehicle {
    protected String brand;
    protected int year;
    
    public void start() {
        System.out.println("Vehicle starting...");
    }
}

// Child class
class Car extends Vehicle {
    private int numberOfDoors;
    
    public Car(String brand, int year, int doors) {
        this.brand = brand;
        this.year = year;
        this.numberOfDoors = doors;
    }
    
    // Override parent method
    @Override
    public void start() {
        System.out.println("Car engine starting...");
    }
    
    // New method
    public void honk() {
        System.out.println("Beep beep!");
    }
}
```

### 3. Polymorphism
**Concept:** "Many forms" - Same method behaves differently in different classes.

**Types:**
- **Compile-time (Method Overloading)**
- **Runtime (Method Overriding)**

```java
// Method Overloading (Compile-time polymorphism)
class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
}

// Method Overriding (Runtime polymorphism)
class Animal {
    public void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

// Usage
Animal myDog = new Dog();
Animal myCat = new Cat();
myDog.makeSound(); // Woof!
myCat.makeSound(); // Meow!
```

---

## 4. Collections Framework

### List vs Set vs Map

| Collection | Ordered | Duplicates | Null Values | Implementation |
|------------|---------|------------|-------------|----------------|
| **List** | Yes | Allowed | Multiple nulls allowed | ArrayList, LinkedList |
| **Set** | No (except LinkedHashSet, TreeSet) | Not allowed | One null allowed (except TreeSet) | HashSet, LinkedHashSet, TreeSet |
| **Map** | No (except LinkedHashMap, TreeMap) | Keys: No, Values: Yes | One null key, multiple null values | HashMap, LinkedHashMap, TreeMap |

### ArrayList vs LinkedList

```java
// ArrayList - Fast random access, slow insert/delete in middle
List<String> arrayList = new ArrayList<>();
arrayList.add("Apple");
arrayList.add("Banana");
arrayList.get(0); // Fast O(1)
arrayList.add(1, "Cherry"); // Slow O(n) for middle insertion

// LinkedList - Slow random access, fast insert/delete in middle
List<String> linkedList = new LinkedList<>();
linkedList.add("Apple");
linkedList.add("Banana");
linkedList.get(0); // Slow O(n)
linkedList.add(1, "Cherry"); // Fast O(1) if iterator is at position
```

### HashSet vs TreeSet vs LinkedHashSet

```java
// HashSet - No order, fastest
Set<String> hashSet = new HashSet<>();
hashSet.add("Banana");
hashSet.add("Apple");
hashSet.add("Cherry");
// Order: Random (Apple, Cherry, Banana)

// TreeSet - Sorted order, slower
Set<String> treeSet = new TreeSet<>();
treeSet.add("Banana");
treeSet.add("Apple");
treeSet.add("Cherry");
// Order: Sorted (Apple, Banana, Cherry)

// LinkedHashSet - Insertion order maintained
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("Banana");
linkedHashSet.add("Apple");
linkedHashSet.add("Cherry");
// Order: Insertion order (Banana, Apple, Cherry)
```

### HashMap vs TreeMap vs LinkedHashMap

```java
// HashMap - No order, fastest, allows one null key
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("Apple", 10);
hashMap.put("Banana", 20);
hashMap.put(null, 30); // Allowed

// TreeMap - Sorted by keys, no null keys
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("Banana", 20);
treeMap.put("Apple", 10);
// Order: Sorted by key (Apple, Banana)

// LinkedHashMap - Insertion order maintained
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put("Banana", 20);
linkedHashMap.put("Apple", 10);
// Order: Insertion order (Banana, Apple)
```

---

## 5. Exception Handling

### try-catch-finally

```java
public class FileProcessor {
    public void readFile(String filename) {
        FileReader reader = null;
        try {
            reader = new FileReader(filename);
            // Read file content
            System.out.println("File opened successfully");
        } catch (FileNotFoundException e) {
            System.err.println("File not found: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("Error reading file: " + e.getMessage());
        } finally {
            // Always executes - cleanup code
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    System.err.println("Error closing file");
                }
            }
        }
    }
}
```

### try-with-resources (Java 7+)

```java
// Automatically closes resources
public void readFileModern(String filename) {
    try (FileReader reader = new FileReader(filename);
         BufferedReader bufferedReader = new BufferedReader(reader)) {
        
        String line;
        while ((line = bufferedReader.readLine()) != null) {
            System.out.println(line);
        }
        
    } catch (IOException e) {
        System.err.println("Error: " + e.getMessage());
    }
    // No need for finally - automatically closed
}
```

### Checked vs Unchecked Exceptions

| Type | Must Handle? | Examples |
|------|-------------|----------|
| **Checked** | Yes (compile-time) | IOException, SQLException, ClassNotFoundException |
| **Unchecked** | No (runtime) | NullPointerException, ArrayIndexOutOfBoundsException, ArithmeticException |

```java
// Checked exception - must handle or declare
public void readFile(String path) throws IOException {
    FileReader reader = new FileReader(path);
}

// Unchecked exception - optional handling
public void divideNumbers(int a, int b) {
    int result = a / b; // May throw ArithmeticException
}
```

---

## 6. String, StringBuilder, StringBuffer

### Differences:

| Feature | String | StringBuilder | StringBuffer |
|---------|--------|---------------|--------------|
| **Mutable** | ❌ No | ✅ Yes | ✅ Yes |
| **Thread-Safe** | ✅ Yes | ❌ No | ✅ Yes |
| **Performance** | Slow (creates new object) | Fast | Slower than StringBuilder |
| **Use When** | Few modifications | Single-threaded, many modifications | Multi-threaded, many modifications |

**Example:**
```java
// String - Immutable
String str = "Hello";
str = str + " World"; // Creates new String object

// StringBuilder - Mutable, not thread-safe, fastest
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // Modifies same object
sb.insert(5, " Beautiful");
sb.delete(5, 15);
String result = sb.toString();

// StringBuffer - Mutable, thread-safe, synchronized
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World"); // Thread-safe
```

---

## 7. Multithreading

### Creating Threads

**Method 1: Extending Thread class**
```java
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + i);
        }
    }
}

// Usage
MyThread thread1 = new MyThread();
thread1.start();
```

**Method 2: Implementing Runnable interface** (Preferred)
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + i);
        }
    }
}

// Usage
Thread thread2 = new Thread(new MyRunnable());
thread2.start();
```

**Method 3: Lambda (Java 8+)**
```java
Thread thread3 = new Thread(() -> {
    for (int i = 0; i < 5; i++) {
        System.out.println(Thread.currentThread().getName() + ": " + i);
    }
});
thread3.start();
```

### Synchronization

```java
class BankAccount {
    private int balance = 1000;
    
    // Synchronized method
    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            System.out.println(Thread.currentThread().getName() + " withdrawing " + amount);
            balance -= amount;
            System.out.println("Balance: " + balance);
        }
    }
    
    // Synchronized block
    public void deposit(int amount) {
        synchronized(this) {
            balance += amount;
            System.out.println("Balance after deposit: " + balance);
        }
    }
}
```

---

## Summary

| Topic | Key Takeaway |
|-------|-------------|
| **Functional Interface** | Interface with one abstract method; enables lambdas |
| **Abstraction** | Hide implementation using abstract classes and interfaces |
| **OOP Concepts** | Encapsulation, Inheritance, Polymorphism |
| **Collections** | List, Set, Map with various implementations |
| **Exception Handling** | try-catch-finally, checked vs unchecked exceptions |
| **String Classes** | String (immutable), StringBuilder (mutable, fast), StringBuffer (thread-safe) |
| **Multithreading** | Thread creation, synchronization for thread safety |
