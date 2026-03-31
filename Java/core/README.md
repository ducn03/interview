# \# ☕ JAVA CORE - INTERVIEW CHEATSHEET

# 

# > \*\*Phiên bản đầy đủ nhất\*\* — đọc phát hiểu luôn, không cần tra thêm  

# > Cập nhật: Java 8–21 | Dùng để ôn interview Backend Java

# 

# \---

# 

# \## 📋 MỤC LỤC

# 

# 1\. \[OOP — 4 Tính Chất Cốt Lõi](#1-oop)

# 2\. \[JVM, JDK, JRE — Khác nhau gì?](#2-jvm-jdk-jre)

# 3\. \[Data Types \& Wrapper Classes](#3-data-types--wrapper-classes)

# 4\. \[String, StringBuilder, StringBuffer](#4-string-stringbuilder-stringbuffer)

# 5\. \[Exception Handling](#5-exception-handling)

# 6\. \[Collections Framework](#6-collections-framework)

# 7\. \[Generics](#7-generics)

# 8\. \[Iterator vs Iterable](#8-iterator-vs-iterable)

# 9\. \[Comparable vs Comparator](#9-comparable-vs-comparator)

# 10\. \[Java 8 — Lambda, Stream, Optional](#10-java-8--lambda-stream-optional)

# 11\. \[Functional Interfaces](#11-functional-interfaces)

# 12\. \[Multithreading \& Concurrency](#12-multithreading--concurrency)

# 13\. \[synchronized, volatile, atomic](#13-synchronized-volatile-atomic)

# 14\. \[Deadlock — Nguyên nhân \& Cách tránh](#14-deadlock)

# 15\. \[Java Memory Model — Heap vs Stack](#15-java-memory-model)

# 16\. \[Garbage Collection](#16-garbage-collection)

# 17\. \[Design Patterns hay gặp](#17-design-patterns)

# 18\. \[SOLID Principles](#18-solid-principles)

# 19\. \[Interface vs Abstract Class](#19-interface-vs-abstract-class)

# 20\. \[Java 9–21 — Tính năng mới quan trọng](#20-java-9-21--tính-năng-mới)

# 

# \---

# 

# \## 1. OOP

# 

# OOP (Object-Oriented Programming) có \*\*4 tính chất\*\* — phỏng vấn hỏi gần như 100%.

# 

# \### 1.1 Encapsulation (Đóng gói)

# 

# > \*\*Che giấu dữ liệu bên trong\*\*, chỉ cho phép truy cập qua `getter/setter`.

# 

# ```java

# public class BankAccount {

# &#x20;   private double balance; // Ẩn field này

# 

# &#x20;   public double getBalance() { return balance; }

# 

# &#x20;   public void deposit(double amount) {

# &#x20;       if (amount > 0) balance += amount; // Kiểm soát logic bên trong

# &#x20;   }

# }

# ```

# 

# \*\*Lý do dùng:\*\*

# \- Ngăn code bên ngoài sửa dữ liệu tùy tiện

# \- Dễ thay đổi implementation mà không ảnh hưởng nơi gọi

# 

# \---

# 

# \### 1.2 Inheritance (Kế thừa)

# 

# > \*\*Class con\*\* thừa hưởng thuộc tính \& phương thức của \*\*class cha\*\*.

# 

# ```java

# public class Animal {

# &#x20;   protected String name;

# 

# &#x20;   public void eat() {

# &#x20;       System.out.println(name + " is eating");

# &#x20;   }

# }

# 

# public class Dog extends Animal {

# &#x20;   public void bark() {

# &#x20;       System.out.println(name + " is barking");

# &#x20;   }

# }

# 

# Dog dog = new Dog();

# dog.name = "Rex";

# dog.eat();  // Kế thừa từ Animal

# dog.bark(); // Của riêng Dog

# ```

# 

# \*\*Lưu ý quan trọng:\*\*

# \- Java chỉ hỗ trợ \*\*single inheritance\*\* (một class chỉ extends 1 class cha)

# \- Dùng `super` để gọi constructor/method của class cha

# \- `final class` không thể bị kế thừa

# 

# \---

# 

# \### 1.3 Polymorphism (Đa hình)

# 

# > Cùng một method nhưng \*\*hành vi khác nhau\*\* tùy vào object thực tế.

# 

# Có 2 loại:

# 

# \*\*Compile-time (Method Overloading):\*\*

# ```java

# public class Calculator {

# &#x20;   public int add(int a, int b) { return a + b; }

# &#x20;   public double add(double a, double b) { return a + b; } // Cùng tên, khác tham số

# &#x20;   public int add(int a, int b, int c) { return a + b + c; }

# }

# ```

# 

# \*\*Runtime (Method Overriding):\*\*

# ```java

# public class Shape {

# &#x20;   public double area() { return 0; }

# }

# 

# public class Circle extends Shape {

# &#x20;   private double radius;

# &#x20;   public Circle(double r) { this.radius = r; }

# 

# &#x20;   @Override

# &#x20;   public double area() { return Math.PI \* radius \* radius; }

# }

# 

# public class Rectangle extends Shape {

# &#x20;   private double w, h;

# &#x20;   public Rectangle(double w, double h) { this.w = w; this.h = h; }

# 

# &#x20;   @Override

# &#x20;   public double area() { return w \* h; }

# }

# 

# // Đây là runtime polymorphism

# Shape s1 = new Circle(5);

# Shape s2 = new Rectangle(4, 6);

# System.out.println(s1.area()); // Gọi Circle.area()

# System.out.println(s2.area()); // Gọi Rectangle.area()

# ```

# 

# \---

# 

# \### 1.4 Abstraction (Trừu tượng)

# 

# > Ẩn chi tiết phức tạp, chỉ \*\*expose những gì cần thiết\*\*.

# 

# Thực hiện qua `abstract class` hoặc `interface`:

# 

# ```java

# public abstract class Vehicle {

# &#x20;   public abstract void startEngine(); // Bắt buộc override

# 

# &#x20;   public void refuel() { // Implement sẵn

# &#x20;       System.out.println("Refueling...");

# &#x20;   }

# }

# 

# public class Car extends Vehicle {

# &#x20;   @Override

# &#x20;   public void startEngine() {

# &#x20;       System.out.println("Car engine starts with key");

# &#x20;   }

# }

# ```

# 

# \---

# 

# \## 2. JVM, JDK, JRE

# 

# | Thành phần | Là gì | Dùng khi nào |

# |------------|-------|--------------|

# | \*\*JDK\*\* (Java Development Kit) | Bộ công cụ phát triển: bao gồm JRE + compiler (`javac`) + tools | Khi \*\*viết code\*\* Java |

# | \*\*JRE\*\* (Java Runtime Environment) | Môi trường chạy: JVM + thư viện chuẩn | Khi chỉ cần \*\*chạy\*\* file `.jar` |

# | \*\*JVM\*\* (Java Virtual Machine) | Máy ảo thực thi bytecode | Chạy ngầm bên trong JRE |

# 

# \*\*Luồng biên dịch:\*\*

# ```

# Source code (.java)

# &#x20;      ↓  javac (compiler)

# &#x20; Bytecode (.class)

# &#x20;      ↓  JVM (interprets/JIT compiles)

# &#x20; Native machine code (chạy trên OS)

# ```

# 

# \*\*JIT Compiler\*\* (Just-In-Time): JVM không interpret từng dòng bytecode mà \*\*compile cả đoạn hay dùng thành native code\*\* để tăng tốc. Đây là lý do Java không chậm như nhiều người nghĩ.

# 

# \---

# 

# \## 3. Data Types \& Wrapper Classes

# 

# \### Primitive Types

# 

# | Type | Size | Default | Range |

# |------|------|---------|-------|

# | `byte` | 1 byte | 0 | -128 → 127 |

# | `short` | 2 bytes | 0 | -32,768 → 32,767 |

# | `int` | 4 bytes | 0 | \~±2.1 tỷ |

# | `long` | 8 bytes | 0L | \~±9.2 \* 10^18 |

# | `float` | 4 bytes | 0.0f | \~7 chữ số thập phân |

# | `double` | 8 bytes | 0.0 | \~15 chữ số thập phân |

# | `char` | 2 bytes | '\\u0000' | 0 → 65,535 |

# | `boolean` | \~1 bit | false | true/false |

# 

# \### Wrapper Classes \& Autoboxing

# 

# ```java

# // Autoboxing: primitive → Wrapper tự động

# Integer x = 42;        // int → Integer

# Double d = 3.14;       // double → Double

# 

# // Unboxing: Wrapper → primitive tự động

# int y = x;             // Integer → int

# 

# // Trap quan trọng khi phỏng vấn:

# Integer a = 127;

# Integer b = 127;

# System.out.println(a == b);  // TRUE — Integer cache -128 to 127

# 

# Integer c = 128;

# Integer d2 = 128;

# System.out.println(c == d2); // FALSE — ngoài cache range, == so sánh reference

# System.out.println(c.equals(d2)); // TRUE — nên dùng equals()

# ```

# 

# \*\*Integer Cache:\*\* Java cache sẵn các Integer từ -128 đến 127 trong pool. Khi dùng `==` với 2 Integer trong range này, chúng cùng trỏ về 1 object → `true`. Ngoài range → 2 object khác nhau → `false`.

# 

# \---

# 

# \## 4. String, StringBuilder, StringBuffer

# 

# \### So sánh

# 

# | Tiêu chí | String | StringBuilder | StringBuffer |

# |----------|--------|---------------|--------------|

# | \*\*Mutability\*\* | Immutable | Mutable | Mutable |

# | \*\*Thread-safe\*\* | Yes (bất biến) | No | Yes |

# | \*\*Performance\*\* | Chậm nếu concat nhiều | Nhanh nhất | Chậm hơn SB vì synchronized |

# | \*\*Storage\*\* | String Pool / Heap | Heap | Heap |

# 

# \### String là Immutable — Tại sao?

# 

# ```java

# String s = "Hello";

# s = s + " World"; // Không sửa object cũ, tạo object mới trên Heap

# // Object "Hello" vẫn còn trong memory (chờ GC)

# ```

# 

# \*\*Lý do thiết kế immutable:\*\*

# 1\. \*\*Security:\*\* tránh thay đổi credentials, path bị thay đổi giữa chừng

# 2\. \*\*Thread-safe:\*\* nhiều thread cùng đọc mà không cần lock

# 3\. \*\*Hashcode caching:\*\* `HashMap` dùng String làm key hiệu quả hơn

# 

# \### String Pool

# 

# ```java

# String a = "hello";          // Lưu trong String Pool

# String b = "hello";          // Tái sử dụng từ pool

# String c = new String("hello"); // Tạo object mới trên Heap

# 

# System.out.println(a == b);       // TRUE  — cùng reference trong pool

# System.out.println(a == c);       // FALSE — c là object khác trên Heap

# System.out.println(a.equals(c));  // TRUE  — so sánh nội dung

# ```

# 

# \### Khi nào dùng gì?

# 

# ```java

# // Dùng String: giá trị ít thay đổi

# String name = "John";

# 

# // Dùng StringBuilder: single-thread, thao tác nhiều

# StringBuilder sb = new StringBuilder();

# for (int i = 0; i < 1000; i++) {

# &#x20;   sb.append("item").append(i).append(",");

# }

# String result = sb.toString();

# 

# // Dùng StringBuffer: multi-thread cần thread-safe

# StringBuffer buffer = new StringBuffer();

# // Nhiều thread cùng append an toàn

# ```

# 

# \---

# 

# \## 5. Exception Handling

# 

# \### Hierarchy

# 

# ```

# Throwable

# ├── Error (không nên catch — JVM level)

# │   ├── OutOfMemoryError

# │   ├── StackOverflowError

# │   └── VirtualMachineError

# └── Exception

# &#x20;   ├── Checked Exception (phải khai báo hoặc try-catch)

# &#x20;   │   ├── IOException

# &#x20;   │   ├── SQLException

# &#x20;   │   └── ClassNotFoundException

# &#x20;   └── Unchecked Exception (RuntimeException — không bắt buộc)

# &#x20;       ├── NullPointerException

# &#x20;       ├── ArrayIndexOutOfBoundsException

# &#x20;       ├── ClassCastException

# &#x20;       ├── IllegalArgumentException

# &#x20;       └── NumberFormatException

# ```

# 

# \### Checked vs Unchecked

# 

# ```java

# // Checked: Compiler bắt buộc xử lý

# public void readFile(String path) throws IOException { // Phải throws hoặc try-catch

# &#x20;   FileReader fr = new FileReader(path);

# }

# 

# // Unchecked: Không bắt buộc (nhưng nên handle)

# public int divide(int a, int b) {

# &#x20;   return a / b; // Có thể throw ArithmeticException nếu b=0

# }

# ```

# 

# \### try-catch-finally

# 

# ```java

# public String readFile(String path) {

# &#x20;   BufferedReader reader = null;

# &#x20;   try {

# &#x20;       reader = new BufferedReader(new FileReader(path));

# &#x20;       return reader.readLine();

# &#x20;   } catch (FileNotFoundException e) {

# &#x20;       System.err.println("File not found: " + path);

# &#x20;       return null;

# &#x20;   } catch (IOException e) {

# &#x20;       System.err.println("Error reading file: " + e.getMessage());

# &#x20;       return null;

# &#x20;   } finally {

# &#x20;       // LUÔN chạy dù có exception hay không — dùng để đóng resource

# &#x20;       if (reader != null) {

# &#x20;           try { reader.close(); } catch (IOException e) { /\* ignore \*/ }

# &#x20;       }

# &#x20;   }

# }

# ```

# 

# \### Try-with-resources (Java 7+) — Cách hiện đại

# 

# ```java

# // AutoCloseable resource tự đóng sau khi ra khỏi block

# public String readFile(String path) throws IOException {

# &#x20;   try (BufferedReader reader = new BufferedReader(new FileReader(path))) {

# &#x20;       return reader.readLine();

# &#x20;   } // reader.close() tự động gọi ở đây

# }

# ```

# 

# \### Custom Exception

# 

# ```java

# public class InsufficientFundsException extends RuntimeException {

# &#x20;   private final double amount;

# 

# &#x20;   public InsufficientFundsException(double amount) {

# &#x20;       super("Insufficient funds: need " + amount + " more");

# &#x20;       this.amount = amount;

# &#x20;   }

# 

# &#x20;   public double getAmount() { return amount; }

# }

# 

# // Sử dụng

# public void withdraw(double amount) {

# &#x20;   if (amount > balance) {

# &#x20;       throw new InsufficientFundsException(amount - balance);

# &#x20;   }

# &#x20;   balance -= amount;

# }

# ```

# 

# \---

# 

# \## 6. Collections Framework

# 

# \### Sơ đồ tổng quan

# 

# ```

# Collection (interface)

# ├── List (ordered, allows duplicates)

# │   ├── ArrayList     — dynamic array, O(1) get, O(n) insert/delete giữa

# │   ├── LinkedList    — doubly linked list, O(n) get, O(1) insert/delete đầu/cuối

# │   └── Vector        — thread-safe ArrayList (cũ, ít dùng)

# ├── Set (no duplicates)

# │   ├── HashSet       — không có thứ tự, O(1) add/contains

# │   ├── LinkedHashSet — giữ insertion order, O(1)

# │   └── TreeSet       — sorted order, O(log n)

# └── Queue / Deque

# &#x20;   ├── PriorityQueue — min-heap, O(log n)

# &#x20;   ├── ArrayDeque    — double-ended queue

# &#x20;   └── LinkedList    — implements Queue \& Deque

# 

# Map (key-value, NOT extends Collection)

# ├── HashMap         — không có thứ tự, O(1) trung bình

# ├── LinkedHashMap   — giữ insertion order

# ├── TreeMap         — sorted by key, O(log n)

# ├── Hashtable       — thread-safe (cũ)

# └── ConcurrentHashMap — thread-safe (hiện đại, nên dùng)

# ```

# 

# \### ArrayList vs LinkedList

# 

# ```java

# // ArrayList — dùng khi thường xuyên get bằng index

# List<String> arrayList = new ArrayList<>();

# arrayList.add("A");

# arrayList.get(0); // O(1)

# arrayList.add(0, "X"); // O(n) — phải shift các phần tử

# 

# // LinkedList — dùng khi thường xuyên insert/delete đầu/cuối

# List<String> linkedList = new LinkedList<>();

# ((LinkedList<String>) linkedList).addFirst("A"); // O(1)

# ((LinkedList<String>) linkedList).addLast("B");  // O(1)

# linkedList.get(5); // O(n) — phải duyệt từ đầu

# ```

# 

# \### HashMap nội bộ hoạt động thế nào?

# 

# ```

# HashMap gồm mảng các "bucket" (Node\[])

# 

# put("key", value):

# 1\. Tính hashCode("key")

# 2\. index = hashCode \& (capacity - 1)  // modulo cho power of 2

# 3\. Nếu bucket\[index] rỗng → lưu trực tiếp

# 4\. Nếu bucket\[index] có phần tử → dùng equals() để kiểm tra

# &#x20;  - Nếu key bằng nhau → update value

# &#x20;  - Nếu key khác nhau → chaining (LinkedList → TreeNode khi ≥ 8 phần tử)

# ```

# 

# ```java

# // Default capacity: 16, load factor: 0.75

# // Khi size > 16 \* 0.75 = 12 → resize lên 32 (x2)

# HashMap<String, Integer> map = new HashMap<>(32, 0.75f); // Custom capacity

# 

# // Các method quan trọng

# map.put("key", 1);

# map.get("key");           // null nếu không tồn tại

# map.getOrDefault("key", 0); // Trả default nếu không có

# map.containsKey("key");

# map.containsValue(1);

# map.putIfAbsent("key", 2); // Chỉ put nếu chưa có key

# map.computeIfAbsent("key", k -> k.length()); // Tính và put nếu chưa có

# 

# // Duyệt Map

# for (Map.Entry<String, Integer> entry : map.entrySet()) {

# &#x20;   System.out.println(entry.getKey() + " = " + entry.getValue());

# }

# ```

# 

# \### HashSet vs TreeSet vs LinkedHashSet

# 

# ```java

# Set<String> hashSet = new HashSet<>();         // Không thứ tự

# Set<String> linkedHashSet = new LinkedHashSet<>(); // Giữ insertion order

# Set<String> treeSet = new TreeSet<>();         // Tự sắp xếp (A,B,C...)

# 

# hashSet.add("banana"); hashSet.add("apple"); hashSet.add("cherry");

# // hashSet print: \[banana, cherry, apple]  — random order

# 

# linkedHashSet.add("banana"); linkedHashSet.add("apple");

# // print: \[banana, apple]  — insertion order

# 

# treeSet.add("banana"); treeSet.add("apple"); treeSet.add("cherry");

# // print: \[apple, banana, cherry]  — sorted

# ```

# 

# \---

# 

# \## 7. Generics

# 

# > \*\*Generics\*\* giúp code tái sử dụng mà vẫn type-safe, tránh ClassCastException.

# 

# ```java

# // Không có generics — dễ lỗi

# List list = new ArrayList();

# list.add("Hello");

# list.add(42);

# String s = (String) list.get(1); // ClassCastException lúc runtime!

# 

# // Có generics — lỗi phát hiện lúc compile

# List<String> strings = new ArrayList<>();

# strings.add("Hello");

# // strings.add(42); // Compile error — an toàn hơn

# String s = strings.get(0); // Không cần cast

# ```

# 

# \### Generic Class \& Method

# 

# ```java

# // Generic class

# public class Pair<A, B> {

# &#x20;   private A first;

# &#x20;   private B second;

# 

# &#x20;   public Pair(A first, B second) {

# &#x20;       this.first = first;

# &#x20;       this.second = second;

# &#x20;   }

# 

# &#x20;   public A getFirst() { return first; }

# &#x20;   public B getSecond() { return second; }

# }

# 

# Pair<String, Integer> pair = new Pair<>("age", 25);

# 

# // Generic method

# public <T extends Comparable<T>> T max(T a, T b) {

# &#x20;   return a.compareTo(b) >= 0 ? a : b;

# }

# ```

# 

# \### Wildcard

# 

# ```java

# // ? extends T — đọc được, không ghi (upper bounded)

# public double sumList(List<? extends Number> list) {

# &#x20;   double sum = 0;

# &#x20;   for (Number n : list) sum += n.doubleValue();

# &#x20;   return sum;

# }

# sumList(new ArrayList<Integer>());  // OK

# sumList(new ArrayList<Double>());   // OK

# 

# // ? super T — ghi được, đọc ra Object (lower bounded)

# public void addNumbers(List<? super Integer> list) {

# &#x20;   list.add(1);

# &#x20;   list.add(2);

# }

# ```

# 

# \---

# 

# \## 8. Iterator vs Iterable

# 

# ```java

# // Iterable: interface có method iterator()

# // — Implement để dùng được trong for-each loop

# public interface Iterable<T> {

# &#x20;   Iterator<T> iterator();

# }

# 

# // Iterator: interface để duyệt collection

# public interface Iterator<T> {

# &#x20;   boolean hasNext();

# &#x20;   T next();

# &#x20;   default void remove() { ... }

# }

# 

# // Ví dụ tự implement

# public class Range implements Iterable<Integer> {

# &#x20;   private int start, end;

# 

# &#x20;   public Range(int start, int end) {

# &#x20;       this.start = start;

# &#x20;       this.end = end;

# &#x20;   }

# 

# &#x20;   @Override

# &#x20;   public Iterator<Integer> iterator() {

# &#x20;       return new Iterator<Integer>() {

# &#x20;           int current = start;

# 

# &#x20;           @Override

# &#x20;           public boolean hasNext() { return current < end; }

# 

# &#x20;           @Override

# &#x20;           public Integer next() { return current++; }

# &#x20;       };

# &#x20;   }

# }

# 

# // Dùng được for-each vì implement Iterable

# for (int i : new Range(1, 5)) {

# &#x20;   System.out.print(i + " "); // 1 2 3 4

# }

# ```

# 

# \---

# 

# \## 9. Comparable vs Comparator

# 

# | | Comparable | Comparator |

# |--|-----------|------------|

# | \*\*Package\*\* | java.lang | java.util |

# | \*\*Method\*\* | `compareTo(T o)` | `compare(T o1, T o2)` |

# | \*\*Nằm trong\*\* | Chính class cần sort | Class riêng biệt |

# | \*\*Dùng khi\*\* | Muốn định nghĩa "natural order" | Muốn nhiều cách sort khác nhau |

# 

# ```java

# // Comparable — natural order (sort mặc định)

# public class Student implements Comparable<Student> {

# &#x20;   private String name;

# &#x20;   private int gpa;

# 

# &#x20;   @Override

# &#x20;   public int compareTo(Student other) {

# &#x20;       return Double.compare(this.gpa, other.gpa); // Sort tăng dần theo GPA

# &#x20;       // Trả về âm: this < other

# &#x20;       // Trả về 0:  this == other

# &#x20;       // Trả về dương: this > other

# &#x20;   }

# }

# 

# List<Student> students = new ArrayList<>();

# Collections.sort(students); // Dùng compareTo

# 

# // Comparator — custom order

# Comparator<Student> byName = Comparator.comparing(s -> s.getName());

# Comparator<Student> byGpaDesc = Comparator.comparingDouble(Student::getGpa).reversed();

# Comparator<Student> complex = Comparator

# &#x20;   .comparingDouble(Student::getGpa).reversed()

# &#x20;   .thenComparing(Student::getName);

# 

# students.sort(byGpaDesc);

# students.sort(complex);

# ```

# 

# \---

# 

# \## 10. Java 8 — Lambda, Stream, Optional

# 

# \### Lambda Expression

# 

# ```java

# // Trước Java 8

# Runnable r = new Runnable() {

# &#x20;   @Override

# &#x20;   public void run() { System.out.println("Hello"); }

# };

# 

# // Java 8 — Lambda

# Runnable r = () -> System.out.println("Hello");

# 

# // So sánh Comparator

# list.sort((a, b) -> a.compareTo(b));

# list.sort(String::compareTo); // Method reference

# ```

# 

# \### Stream API — Cực kỳ hay hỏi

# 

# ```java

# List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

# 

# // Lọc số chẵn, nhân đôi, collect

# List<Integer> result = numbers.stream()

# &#x20;   .filter(n -> n % 2 == 0)   // intermediate: giữ lại số chẵn

# &#x20;   .map(n -> n \* 2)            // intermediate: nhân đôi

# &#x20;   .collect(Collectors.toList()); // terminal: kết thúc stream

# // \[4, 8, 12, 16, 20]

# 

# // reduce — tính tổng

# int sum = numbers.stream()

# &#x20;   .reduce(0, Integer::sum);  // 55

# 

# // Tìm max

# Optional<Integer> max = numbers.stream()

# &#x20;   .max(Integer::compareTo);

# max.ifPresent(System.out::println); // 10

# 

# // Group by

# Map<Integer, List<String>> byLength = Stream.of("apple", "banana", "kiwi", "fig", "mango")

# &#x20;   .collect(Collectors.groupingBy(String::length));

# // {3=\[fig], 4=\[kiwi], 5=\[apple, mango], 6=\[banana]}

# 

# // FlatMap — flatten nested list

# List<List<Integer>> nested = Arrays.asList(

# &#x20;   Arrays.asList(1, 2), Arrays.asList(3, 4), Arrays.asList(5)

# );

# List<Integer> flat = nested.stream()

# &#x20;   .flatMap(Collection::stream)

# &#x20;   .collect(Collectors.toList());

# // \[1, 2, 3, 4, 5]

# ```

# 

# \*\*Intermediate vs Terminal operations:\*\*

# \- \*\*Intermediate\*\* (lazy, không chạy ngay): `filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`, `skip`, `peek`

# \- \*\*Terminal\*\* (trigger execution): `collect`, `forEach`, `reduce`, `count`, `findFirst`, `anyMatch`, `allMatch`, `noneMatch`, `min`, `max`, `toArray`

# 

# \### Optional — Tránh NullPointerException

# 

# ```java

# // Tạo Optional

# Optional<String> empty = Optional.empty();

# Optional<String> value = Optional.of("hello");      // Throw NPE nếu null

# Optional<String> nullable = Optional.ofNullable(getName()); // OK nếu null

# 

# // Sử dụng

# nullable.isPresent();               // Check có value không

# nullable.get();                     // Lấy giá trị (throw nếu empty)

# nullable.orElse("default");         // Lấy hoặc trả default

# nullable.orElseGet(() -> compute()); // Lazy evaluation

# nullable.orElseThrow(() -> new NotFoundException()); // Throw nếu empty

# nullable.map(String::toUpperCase);  // Transform nếu có value

# nullable.filter(s -> s.length() > 3); // Filter

# 

# // Chain thực tế

# Optional<String> city = Optional.ofNullable(user)

# &#x20;   .map(User::getAddress)

# &#x20;   .map(Address::getCity)

# &#x20;   .filter(c -> !c.isEmpty());

# ```

# 

# \---

# 

# \## 11. Functional Interfaces

# 

# > Interface có \*\*đúng 1 abstract method\*\* — dùng được với Lambda.

# 

# ```java

# // 4 Functional Interface phổ biến nhất

# // 1. Predicate<T>: T → boolean

# Predicate<String> isLong = s -> s.length() > 5;

# isLong.test("Hello");   // false

# isLong.test("Hello World"); // true

# isLong.and(s -> s.startsWith("H")); // Kết hợp

# 

# // 2. Function<T, R>: T → R

# Function<String, Integer> length = String::length;

# length.apply("Hello"); // 5

# Function<Integer, Integer> doubler = x -> x \* 2;

# length.andThen(doubler).apply("Hello"); // 10

# 

# // 3. Consumer<T>: T → void

# Consumer<String> print = System.out::println;

# print.accept("Hello"); // In ra Hello

# 

# // 4. Supplier<T>: () → T

# Supplier<String> greeting = () -> "Hello World";

# greeting.get(); // "Hello World"

# 

# // 5. BiFunction<T, U, R>: (T, U) → R

# BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);

# repeat.apply("ab", 3); // "ababab"

# ```

# 

# \---

# 

# \## 12. Multithreading \& Concurrency

# 

# \### Tạo Thread

# 

# ```java

# // Cách 1: Extends Thread

# class MyThread extends Thread {

# &#x20;   @Override

# &#x20;   public void run() {

# &#x20;       System.out.println("Thread: " + getName());

# &#x20;   }

# }

# new MyThread().start();

# 

# // Cách 2: Implement Runnable (Recommended — không chiếm slot extends)

# class MyTask implements Runnable {

# &#x20;   @Override

# &#x20;   public void run() {

# &#x20;       System.out.println("Task running");

# &#x20;   }

# }

# new Thread(new MyTask()).start();

# Thread t = new Thread(() -> System.out.println("Lambda thread"));

# t.start();

# 

# // Cách 3: Callable + Future — có thể trả về giá trị và throw exception

# Callable<Integer> task = () -> {

# &#x20;   Thread.sleep(1000);

# &#x20;   return 42;

# };

# ExecutorService executor = Executors.newFixedThreadPool(4);

# Future<Integer> future = executor.submit(task);

# Integer result = future.get(); // Block cho đến khi xong

# executor.shutdown();

# ```

# 

# \### Thread Lifecycle

# 

# ```

# NEW → RUNNABLE → RUNNING → TERMINATED

# &#x20;                   ↓ ↑

# &#x20;                BLOCKED/WAITING/TIMED\_WAITING

# ```

# 

# \### ExecutorService — Cách hiện đại

# 

# ```java

# // Không nên tự tạo Thread bằng tay trong production

# // Dùng ExecutorService để quản lý thread pool

# 

# ExecutorService pool = Executors.newFixedThreadPool(4); // 4 threads cố định

# ExecutorService cached = Executors.newCachedThreadPool(); // Tự scale

# ExecutorService single = Executors.newSingleThreadExecutor(); // 1 thread

# 

# // Submit task

# pool.execute(() -> System.out.println("fire and forget"));

# Future<String> f = pool.submit(() -> "result");

# 

# // Shutdown gracefully

# pool.shutdown(); // Không nhận task mới, chờ task cũ xong

# pool.shutdownNow(); // Cố gắng stop ngay lập tức

# 

# // Java 19+: Virtual Threads

# ExecutorService vt = Executors.newVirtualThreadPerTaskExecutor();

# ```

# 

# \---

# 

# \## 13. synchronized, volatile, atomic

# 

# \### synchronized

# 

# ```java

# // Instance method lock — lock trên this object

# public synchronized void increment() {

# &#x20;   count++;

# }

# 

# // Static method lock — lock trên Class object

# public static synchronized void staticMethod() { }

# 

# // Synchronized block — giảm scope lock để tăng performance

# public void method() {

# &#x20;   // Code không cần lock

# &#x20;   synchronized (this) {

# &#x20;       count++; // Chỉ lock đoạn này

# &#x20;   }

# &#x20;   // Code không cần lock

# }

# 

# // Lock trên object riêng

# private final Object lock = new Object();

# public void method() {

# &#x20;   synchronized (lock) { ... }

# }

# ```

# 

# \### volatile

# 

# ```java

# // volatile đảm bảo mọi thread đọc giá trị mới nhất từ RAM

# // Không dùng cache của CPU (L1/L2 cache)

# private volatile boolean running = true;

# 

# // Thread 1

# public void stop() { running = false; } // Ghi vào RAM ngay

# 

# // Thread 2

# public void run() {

# &#x20;   while (running) { // Đọc từ RAM thay vì cache

# &#x20;       doWork();

# &#x20;   }

# }

# ```

# 

# \*\*Khi nào dùng volatile?\*\*

# \- Chỉ 1 thread ghi, nhiều thread đọc

# \- Không có compound operations (i++ KHÔNG an toàn với volatile vì thực ra là: read → increment → write)

# 

# \### Atomic Classes

# 

# ```java

# // Dùng CAS (Compare-And-Swap) — không cần lock, nhanh hơn synchronized

# AtomicInteger count = new AtomicInteger(0);

# count.incrementAndGet(); // i++ an toàn

# count.addAndGet(5);

# count.compareAndSet(5, 10); // Nếu value == 5 thì set thành 10

# 

# AtomicBoolean flag = new AtomicBoolean(false);

# flag.compareAndSet(false, true); // Atomic check-and-set

# 

# AtomicReference<String> ref = new AtomicReference<>("old");

# ref.compareAndSet("old", "new");

# ```

# 

# \---

# 

# \## 14. Deadlock

# 

# \### Deadlock là gì?

# 

# > Xảy ra khi \*\*2 thread giữ lock và chờ lock của nhau\*\* mãi mãi.

# 

# ```java

# // Deadlock kinh điển

# Object lock1 = new Object();

# Object lock2 = new Object();

# 

# Thread t1 = new Thread(() -> {

# &#x20;   synchronized (lock1) {

# &#x20;       sleep(100); // Giữ lock1, chờ lock2

# &#x20;       synchronized (lock2) { ... }

# &#x20;   }

# });

# 

# Thread t2 = new Thread(() -> {

# &#x20;   synchronized (lock2) {

# &#x20;       sleep(100); // Giữ lock2, chờ lock1

# &#x20;       synchronized (lock1) { ... }

# &#x20;   }

# });

# ```

# 

# \### 4 điều kiện dẫn đến Deadlock (Coffman Conditions)

# 

# 1\. \*\*Mutual Exclusion\*\* — Resource chỉ 1 thread dùng tại một thời điểm

# 2\. \*\*Hold and Wait\*\* — Thread giữ resource và chờ thêm resource khác

# 3\. \*\*No Preemption\*\* — Không ai có thể cưỡng bức lấy resource từ thread

# 4\. \*\*Circular Wait\*\* — Thread A chờ B, B chờ C, C chờ A

# 

# \### Cách tránh Deadlock

# 

# ```java

# // Giải pháp 1: Lock theo thứ tự nhất quán

# // Cả t1 và t2 đều lock theo thứ tự: lock1 trước, lock2 sau

# Thread t1 = new Thread(() -> {

# &#x20;   synchronized (lock1) {

# &#x20;       synchronized (lock2) { ... }

# &#x20;   }

# });

# Thread t2 = new Thread(() -> {

# &#x20;   synchronized (lock1) { // Cùng thứ tự với t1

# &#x20;       synchronized (lock2) { ... }

# &#x20;   }

# });

# 

# // Giải pháp 2: tryLock với timeout

# ReentrantLock l1 = new ReentrantLock();

# ReentrantLock l2 = new ReentrantLock();

# 

# if (l1.tryLock(1, TimeUnit.SECONDS)) {

# &#x20;   try {

# &#x20;       if (l2.tryLock(1, TimeUnit.SECONDS)) {

# &#x20;           try { // Làm việc }

# &#x20;           finally { l2.unlock(); }

# &#x20;       }

# &#x20;   } finally { l1.unlock(); }

# }

# ```

# 

# \---

# 

# \## 15. Java Memory Model

# 

# \### Heap vs Stack

# 

# | | Stack | Heap |

# |--|-------|------|

# | \*\*Lưu gì\*\* | Primitive values, object references, call frames | Objects, arrays |

# | \*\*Quản lý\*\* | LIFO tự động | GC quản lý |

# | \*\*Thread\*\* | Mỗi thread có stack riêng | Chia sẻ giữa tất cả thread |

# | \*\*Kích thước\*\* | Nhỏ (\~512KB - 8MB) | Lớn (GBs) |

# | \*\*Tốc độ\*\* | Nhanh hơn | Chậm hơn |

# | \*\*Error\*\* | StackOverflowError | OutOfMemoryError |

# 

# ```java

# public void example() {

# &#x20;   int x = 10;                  // Stack: biến primitive

# &#x20;   String s = "hello";          // Stack: reference s, Heap: object "hello"

# &#x20;   Person p = new Person("John"); // Stack: reference p, Heap: Person object

# }

# // Khi method kết thúc: x, s, p bị pop khỏi stack

# // Object "hello", Person vẫn ở heap cho đến khi GC dọn

# ```

# 

# \### Heap Generations

# 

# ```

# Heap

# ├── Young Generation

# │   ├── Eden Space — Object mới sinh ra ở đây

# │   ├── Survivor S0

# │   └── Survivor S1

# └── Old Generation (Tenured)

# &#x20;   └── Objects sống sót qua nhiều GC cycles

# ```

# 

# \---

# 

# \## 16. Garbage Collection

# 

# \### GC hoạt động thế nào?

# 

# 1\. \*\*Mark\*\* — Đánh dấu tất cả object còn được reference (reachable)

# 2\. \*\*Sweep\*\* — Xóa các object không được đánh dấu

# 3\. \*\*Compact\*\* — Dồn memory lại, loại bỏ fragmentation

# 

# \### GC Algorithms

# 

# | Algorithm | Đặc điểm | Dùng khi |

# |-----------|----------|----------|

# | \*\*Serial GC\*\* | Single-thread, dừng ứng dụng | App nhỏ, single-core |

# | \*\*Parallel GC\*\* | Multi-thread, vẫn dừng | Throughput là ưu tiên |

# | \*\*G1 GC\*\* | Chia heap thành regions, pause ngắn | Default Java 9+, balanced |

# | \*\*ZGC\*\* | Pause < 1ms | Low-latency app (Java 15+) |

# | \*\*Shenandoah\*\* | Pause < 10ms | Low-latency (OpenJDK) |

# 

# \### Cách trigger GC

# 

# ```java

# System.gc(); // Gợi ý JVM chạy GC — không đảm bảo chạy ngay

# Runtime.getRuntime().gc();

# ```

# 

# \### Strong vs Weak vs Soft vs Phantom References

# 

# ```java

# // Strong Reference — GC không bao giờ thu hồi

# String s = "hello"; // strong reference

# 

# // Weak Reference — GC có thể thu hồi bất cứ lúc nào

# WeakReference<String> weak = new WeakReference<>(new String("hello"));

# String val = weak.get(); // Có thể null nếu GC đã thu hồi

# 

# // Soft Reference — GC chỉ thu hồi khi sắp hết memory

# SoftReference<byte\[]> cache = new SoftReference<>(new byte\[1024]);

# // Dùng cho cache

# 

# // PhantomReference — dùng để cleanup sau khi GC (hiếm dùng)

# ```

# 

# \---

# 

# \## 17. Design Patterns

# 

# \### Singleton

# 

# ```java

# // Thread-safe Singleton với double-checked locking

# public class Singleton {

# &#x20;   private static volatile Singleton instance;

# 

# &#x20;   private Singleton() {}

# 

# &#x20;   public static Singleton getInstance() {

# &#x20;       if (instance == null) {                     // Check 1: không lock

# &#x20;           synchronized (Singleton.class) {

# &#x20;               if (instance == null) {             // Check 2: trong lock

# &#x20;                   instance = new Singleton();

# &#x20;               }

# &#x20;           }

# &#x20;       }

# &#x20;       return instance;

# &#x20;   }

# }

# 

# // Cách đơn giản hơn: Enum Singleton (Bill Pugh recommend)

# public enum Singleton {

# &#x20;   INSTANCE;

# &#x20;   public void doSomething() { ... }

# }

# Singleton.INSTANCE.doSomething();

# ```

# 

# \### Builder

# 

# ```java

# public class Person {

# &#x20;   private final String name;

# &#x20;   private final int age;

# &#x20;   private final String email;

# 

# &#x20;   private Person(Builder builder) {

# &#x20;       this.name = builder.name;

# &#x20;       this.age = builder.age;

# &#x20;       this.email = builder.email;

# &#x20;   }

# 

# &#x20;   public static class Builder {

# &#x20;       private String name;

# &#x20;       private int age;

# &#x20;       private String email;

# 

# &#x20;       public Builder name(String name) { this.name = name; return this; }

# &#x20;       public Builder age(int age) { this.age = age; return this; }

# &#x20;       public Builder email(String email) { this.email = email; return this; }

# &#x20;       public Person build() { return new Person(this); }

# &#x20;   }

# }

# 

# Person p = new Person.Builder()

# &#x20;   .name("John").age(25).email("john@example.com").build();

# ```

# 

# \### Factory Method

# 

# ```java

# public interface Notification {

# &#x20;   void send(String message);

# }

# 

# public class EmailNotification implements Notification {

# &#x20;   public void send(String message) { System.out.println("Email: " + message); }

# }

# 

# public class SMSNotification implements Notification {

# &#x20;   public void send(String message) { System.out.println("SMS: " + message); }

# }

# 

# public class NotificationFactory {

# &#x20;   public static Notification create(String type) {

# &#x20;       return switch (type) {

# &#x20;           case "email" -> new EmailNotification();

# &#x20;           case "sms" -> new SMSNotification();

# &#x20;           default -> throw new IllegalArgumentException("Unknown: " + type);

# &#x20;       };

# &#x20;   }

# }

# 

# Notification n = NotificationFactory.create("email");

# n.send("Hello!");

# ```

# 

# \### Observer

# 

# ```java

# // EventEmitter pattern

# public interface Observer {

# &#x20;   void update(String event, Object data);

# }

# 

# public class EventBus {

# &#x20;   private Map<String, List<Observer>> listeners = new HashMap<>();

# 

# &#x20;   public void subscribe(String event, Observer observer) {

# &#x20;       listeners.computeIfAbsent(event, k -> new ArrayList<>()).add(observer);

# &#x20;   }

# 

# &#x20;   public void publish(String event, Object data) {

# &#x20;       listeners.getOrDefault(event, Collections.emptyList())

# &#x20;           .forEach(o -> o.update(event, data));

# &#x20;   }

# }

# ```

# 

# \---

# 

# \## 18. SOLID Principles

# 

# | Chữ | Nguyên tắc | Một câu tóm tắt |

# |-----|-----------|-----------------|

# | \*\*S\*\* | Single Responsibility | Một class chỉ có một lý do để thay đổi |

# | \*\*O\*\* | Open/Closed | Mở để extend, đóng để modify |

# | \*\*L\*\* | Liskov Substitution | Class con phải thay thế được class cha |

# | \*\*I\*\* | Interface Segregation | Nhiều interface nhỏ hơn 1 interface lớn |

# | \*\*D\*\* | Dependency Inversion | Depend on abstractions, not concretions |

# 

# ```java

# // D — Dependency Inversion (hay gặp nhất trong interview)

# // BAD: depend on concrete class

# public class OrderService {

# &#x20;   private MySQLRepository repo = new MySQLRepository(); // Tightly coupled

# }

# 

# // GOOD: depend on interface

# public class OrderService {

# &#x20;   private final OrderRepository repo; // Interface

# 

# &#x20;   public OrderService(OrderRepository repo) { // Inject dependency

# &#x20;       this.repo = repo;

# &#x20;   }

# }

# // Giờ có thể inject MySQLRepository, MongoRepository, MockRepository...

# ```

# 

# \---

# 

# \## 19. Interface vs Abstract Class

# 

# | Tiêu chí | Interface | Abstract Class |

# |----------|-----------|----------------|

# | \*\*Multiple inheritance\*\* | ✅ Implement nhiều | ❌ Extends 1 |

# | \*\*Constructor\*\* | ❌ | ✅ |

# | \*\*Fields\*\* | `public static final` chỉ | Mọi loại |

# | \*\*Methods\*\* | `abstract`, `default`, `static` | `abstract` + implemented |

# | \*\*Access modifiers\*\* | Mặc định `public` | Mọi modifier |

# | \*\*Dùng khi\*\* | Định nghĩa contract (IS-A từ ngoài) | Chia sẻ implementation (IS-A từ trong) |

# 

# ```java

# // Interface — "Can do" contract

# public interface Flyable {

# &#x20;   void fly();

# &#x20;   default void land() { System.out.println("Landing..."); } // Java 8+

# &#x20;   static Flyable noOp() { return () -> {}; } // Static factory

# }

# 

# // Abstract class — "Is a" base

# public abstract class Animal {

# &#x20;   protected String name;

# 

# &#x20;   public Animal(String name) { this.name = name; }

# 

# &#x20;   public abstract void makeSound(); // Bắt buộc override

# 

# &#x20;   public void breathe() { System.out.println("Breathing..."); } // Dùng chung

# }

# 

# // Kết hợp cả hai

# public class Eagle extends Animal implements Flyable {

# &#x20;   public Eagle(String name) { super(name); }

# 

# &#x20;   @Override

# &#x20;   public void makeSound() { System.out.println("Screech!"); }

# 

# &#x20;   @Override

# &#x20;   public void fly() { System.out.println(name + " is soaring"); }

# }

# ```

# 

# \---

# 

# \## 20. Java 9–21 — Tính năng mới

# 

# \### Java 9 — Module System

# ```java

# // module-info.java

# module com.myapp.service {

# &#x20;   requires java.sql;

# &#x20;   exports com.myapp.service.api;

# }

# ```

# 

# \### Java 10 — var (Local Variable Type Inference)

# ```java

# var list = new ArrayList<String>(); // Infer là ArrayList<String>

# var map = new HashMap<String, Integer>();

# // Chỉ dùng được trong local scope, không dùng cho field/param

# ```

# 

# \### Java 14–16 — Records

# ```java

# // Tự generate: constructor, getters, equals, hashCode, toString

# public record Point(int x, int y) { }

# 

# Point p = new Point(1, 2);

# p.x(); // getter

# p.y();

# ```

# 

# \### Java 14–17 — Sealed Classes

# ```java

# public sealed class Shape permits Circle, Rectangle, Triangle { }

# public final class Circle extends Shape { ... }

# public final class Rectangle extends Shape { ... }

# // Chỉ các class được permits mới có thể extends Shape

# ```

# 

# \### Java 16+ — Pattern Matching instanceof

# ```java

# // Cũ

# if (obj instanceof String) {

# &#x20;   String s = (String) obj;

# &#x20;   System.out.println(s.length());

# }

# 

# // Mới — kết hợp check và cast

# if (obj instanceof String s) {

# &#x20;   System.out.println(s.length());

# }

# ```

# 

# \### Java 17+ — Switch Expressions

# ```java

# String result = switch (day) {

# &#x20;   case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";

# &#x20;   case SATURDAY, SUNDAY -> "Weekend";

# };

# ```

# 

# \### Java 19–21 — Virtual Threads (Project Loom)

# ```java

# // Virtual thread cực nhẹ — triệu thread mà không tốn nhiều memory

# Thread vt = Thread.ofVirtual().start(() -> {

# &#x20;   System.out.println("Virtual thread!");

# });

# 

# // Dùng executor

# try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {

# &#x20;   IntStream.range(0, 1\_000\_000).forEach(i ->

# &#x20;       executor.submit(() -> task(i))

# &#x20;   );

# }

# ```

# 

# \### Java 21 — Sequenced Collections

# ```java

# // Interface mới: SequencedCollection, SequencedMap

# List<String> list = new ArrayList<>(List.of("a", "b", "c"));

# list.getFirst(); // "a"

# list.getLast();  // "c"

# list.reversed(); // \[c, b, a]

# ```

# 

# \---

# 

# \## 📌 Quick Reference — Câu hỏi hay gặp

# 

# | Câu hỏi | Trả lời nhanh |

# |---------|---------------|

# | `==` vs `equals()` | `==` so sánh reference; `equals()` so sánh nội dung |

# | `final` keyword | Biến: không đổi giá trị; Method: không override; Class: không extends |

# | `static` keyword | Thuộc về class, không phải instance |

# | `this` vs `super` | `this` = object hiện tại; `super` = class cha |

# | Pass by value hay reference? | Java \*\*luôn pass by value\*\* — với object thì pass copy của reference |

# | `abstract` vs `interface` | Abstract: IS-A, có state; Interface: CAN-DO, contract |

# | Khi nào dùng HashMap vs TreeMap? | HashMap: O(1), không cần thứ tự; TreeMap: O(log n), cần sort |

# | Thread vs Runnable | Runnable linh hoạt hơn vì không chiếm slot `extends` |

# | `wait()` vs `sleep()` | `wait()`: release lock, dùng trong synchronized; `sleep()`: giữ lock |

# | String immutability | Security, thread-safe, hashcode caching, String pool |

# 

# \---

# 

# \*README này cover \~90% câu hỏi Java Core trong interview — chúc bạn phỏng vấn thuận lợi! 🚀\*

