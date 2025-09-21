---
title: "5 Sai lầm phổ biến khi học Java mà mình đã mắc phải"
date: 2025-09-18
tags: ["java", "sai-lam", "kinh-nghiem", "hoc-tap"]
categories: ["Java"]
author: "Tên bạn"
description: "Những sai lầm cơ bản khi học Java và cách khắc phục chúng qua kinh nghiệm thực tế"
featuredImage: "/images/featured.jpg"
---

Xin chào các bạn! Hôm nay mình muốn chia sẻ về những sai lầm "kinh điển" mà hầu như ai học Java cũng mắc phải, bao gồm cả mình. Hy vọng qua bài viết này, các bạn newbie sẽ tránh được những "hố đen" mà mình đã rơi vào.

## Sai lầm #1: Lạm dụng `public static void main`

### Mình từng làm thế này:

```java
public class Calculator {
    public static void main(String[] args) {
        // Viết tất cả logic trong main
        int a = 10, b = 5;
        int sum = a + b;
        int diff = a - b;
        int product = a * b;
        int quotient = a / b;

        System.out.println("Sum: " + sum);
        System.out.println("Difference: " + diff);
        // ... và còn 50 dòng code nữa
    }
}
```

### Vấn đề:

- Code không thể tái sử dụng
- Khó debug và maintain
- Không theo nguyên tắc OOP

### Cách khắc phục:

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println("Sum: " + calc.add(10, 5));
        System.out.println("Difference: " + calc.subtract(10, 5));
    }
}
```

## Sai lầm #2: Không hiểu về Reference vs Value

### Mình từng nghĩ:

```java
public class Student {
    String name;

    public static void main(String[] args) {
        Student s1 = new Student();
        s1.name = "Nam";

        Student s2 = s1; // Mình tưởng s2 là bản copy của s1
        s2.name = "Linh";

        System.out.println(s1.name); // Tại sao lại in ra "Linh"???
    }
}
```

### Bài học:

Trong Java, object được truyền theo reference, không phải value. Khi gán `s2 = s1`, cả hai đều trỏ đến cùng một object trong memory.

### Giải pháp:

```java
public class Student implements Cloneable {
    String name;

    public Student(String name) {
        this.name = name;
    }

    // Constructor copy
    public Student(Student other) {
        this.name = other.name;
    }

    public static void main(String[] args) {
        Student s1 = new Student("Nam");
        Student s2 = new Student(s1); // Tạo object mới
        s2.name = "Linh";

        System.out.println(s1.name); // "Nam"
        System.out.println(s2.name); // "Linh"
    }
}
```

## Sai lầm #3: Không xử lý Exception đúng cách

### Code "tệ" của mình:

```java
public void readFile(String fileName) {
    try {
        FileReader file = new FileReader(fileName);
        BufferedReader reader = new BufferedReader(file);
        String line = reader.readLine();
        System.out.println(line);
    } catch (Exception e) {
        // Im lặng, không làm gì cả - RỐT THẬM TỆ!
    }
}
```

### Vấn đề:

- "Nuốt" exception mà không xử lý
- Sử dụng `Exception` quá general
- Không đóng resource

### Cách đúng:

```java
public void readFile(String fileName) {
    try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
        String line = reader.readLine();
        if (line != null) {
            System.out.println(line);
        }
    } catch (FileNotFoundException e) {
        System.err.println("Không tìm thấy file: " + fileName);
        // Log hoặc throw lại exception
    } catch (IOException e) {
        System.err.println("Lỗi đọc file: " + e.getMessage());
    }
}
```

## Sai lầm #4: So sánh String bằng `==`

### Code sai:

```java
public class StringComparison {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = new String("Hello");

        if (s1 == s2) {  // SAAAIIII!
            System.out.println("Equal");
        } else {
            System.out.println("Not equal"); // Sẽ in ra này
        }
    }
}
```

### Giải thích:

- `==` so sánh reference, không phải content
- `s1` trỏ đến String Pool
- `s2` trỏ đến object mới trong heap

### Cách đúng:

```java
public class StringComparison {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = new String("Hello");

        if (s1.equals(s2)) {  // So sánh content
            System.out.println("Equal");
        }

        // Hoặc an toàn hơn với null check
        if (Objects.equals(s1, s2)) {
            System.out.println("Equal and null-safe");
        }
    }
}
```

## Sai lầm #5: Không hiểu về Access Modifier

### Code của mình lúc trước:

```java
public class BankAccount {
    public double balance; // Ai cũng có thể truy cập!

    public BankAccount(double initialBalance) {
        balance = initialBalance;
    }
}

// Ở nơi khác
BankAccount account = new BankAccount(1000);
account.balance = -500; // Oops! Balance âm???
```

### Vấn đề:

- Dữ liệu quan trọng không được bảo vệ
- Vi phạm nguyên tắc Encapsulation

### Cách khắc phục:

```java
public class BankAccount {
    private double balance; // Chỉ class này truy cập được

    public BankAccount(double initialBalance) {
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        } else {
            throw new IllegalArgumentException("Balance không thể âm");
        }
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
}
```

## Tổng kết

Những sai lầm này đã giúp mình hiểu sâu hơn về Java. Đừng sợ mắc lỗi, quan trọng là học hỏi từ chúng!

### Tips để tránh sai lầm:

1. **Practice coding every day** - Luyện tập thường xuyên
2. **Read other people's code** - Đọc code của người khác
3. **Use IDE effectively** - Tận dụng IDE để catch lỗi sớm
4. **Code review** - Nhờ bạn bè review code
5. **Read documentation** - Đọc docs chính thức của Java

Các bạn đã mắc phải sai lầm nào trong danh sách này chưa? Hoặc có sai lầm nào khác muốn chia sẻ? Comment bên dưới nhé!

---

_Keep learning, keep coding! ☕_
