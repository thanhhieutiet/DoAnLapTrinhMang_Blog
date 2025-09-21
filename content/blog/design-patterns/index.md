---
title: "Design Patterns trong Java vs JavaScript: Cùng bài toán, khác cách giải"
date: 2025-10-01
tags: ["java", "javascript", "design-patterns", "oop", "architecture"]
categories: ["Software Design"]
author: "Tên bạn"
description: "So sánh cách implement các Design Patterns phổ biến trong Java và JavaScript"
featuredImage: "/images/featured.jpg"
---

Chào mọi người! Design Patterns là một chủ đề mà mình nghĩ rất quan trọng nhưng lại khá dry khi học lý thuyết. Hôm nay mình muốn chia sẻ cách implement một số patterns phổ biến trong cả Java và JavaScript, qua những ví dụ thực tế mà mình đã áp dụng trong các dự án.

## 1. Singleton Pattern - "Chỉ có một!"

### Bài toán thực tế:

Mình cần một Database Connection Manager, đảm bảo chỉ có một instance duy nhất trong toàn bộ app.

### Java Implementation:

```java
// Thread-safe Singleton với lazy initialization
public class DatabaseManager {
    private static volatile DatabaseManager instance;
    private Connection connection;

    private DatabaseManager() {
        // Private constructor ngăn instantiate từ bên ngoài
        this.connection = createConnection();
    }

    public static DatabaseManager getInstance() {
        if (instance == null) {
            synchronized (DatabaseManager.class) {
                if (instance == null) {
                    instance = new DatabaseManager();
                }
            }
        }
        return instance;
    }

    public Connection getConnection() {
        if (connection == null || connection.isClosed()) {
            connection = createConnection();
        }
        return connection;
    }

    private Connection createConnection() {
        try {
            return DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/mydb",
                "username",
                "password"
            );
        } catch (SQLException e) {
            throw new RuntimeException("Cannot create database connection", e);
        }
    }
}

// Sử dụng
DatabaseManager dbManager = DatabaseManager.getInstance();
Connection conn = dbManager.getConnection();
```

### JavaScript Implementation:

```javascript
// ES6 Module - natural singleton
class DatabaseManager {
  constructor() {
    if (DatabaseManager.instance) {
      return DatabaseManager.instance;
    }

    this.connection = null;
    DatabaseManager.instance = this;
    return this;
  }

  async getConnection() {
    if (!this.connection) {
      this.connection = await this.createConnection();
    }
    return this.connection;
  }

  async createConnection() {
    const mysql = require("mysql2/promise");
    try {
      return await mysql.createConnection({
        host: "localhost",
        user: "username",
        password: "password",
        database: "mydb",
      });
    } catch (error) {
      throw new Error(`Cannot create database connection: ${error.message}`);
    }
  }
}

// Export instance - Module singleton
const databaseManager = new DatabaseManager();
module.exports = databaseManager;

// Hoặc cách đơn giản hơn với Object literal
const DatabaseManager = {
  connection: null,

  async getConnection() {
    if (!this.connection) {
      this.connection = await this.createConnection();
    }
    return this.connection;
  },

  async createConnection() {
    // Implementation...
  },
};

module.exports = DatabaseManager;
```

**Nhận xét:** JavaScript linh hoạt hơn, có thể dùng module system để tạo natural singleton. Java cần handle thread-safety cẩn thận hơn.

## 2. Factory Pattern - "Nhà máy sản xuất objects"

### Bài toán:

Tạo different types của notifications (Email, SMS, Push) based on user preferences.

### Java Implementation:

```java
// Abstract Product
public abstract class Notification {
    public abstract void send(String message, String recipient);
}

// Concrete Products
public class EmailNotification extends Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending email to " + recipient + ": " + message);
        // Email sending logic
    }
}

public class SMSNotification extends Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
        // SMS sending logic
    }
}

public class PushNotification extends Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending push notification to " + recipient + ": " + message);
        // Push notification logic
    }
}

// Factory
public class NotificationFactory {
    public enum NotificationType {
        EMAIL, SMS, PUSH
    }

    public static Notification createNotification(NotificationType type) {
        switch (type) {
            case EMAIL:
                return new EmailNotification();
            case SMS:
                return new SMSNotification();
            case PUSH:
                return new PushNotification();
            default:
                throw new IllegalArgumentException("Unknown notification type: " + type);
        }
    }
}

// Sử dụng
public class NotificationService {
    public void sendNotification(String type, String message, String recipient) {
        NotificationType notificationType = NotificationType.valueOf(type.toUpperCase());
        Notification notification = NotificationFactory.createNotification(notificationType);
        notification.send(message, recipient);
    }
}
```

### JavaScript Implementation:

```javascript
// Base class (không bắt buộc trong JS)
class Notification {
  send(message, recipient) {
    throw new Error("send() method must be implemented");
  }
}

// Concrete implementations
class EmailNotification extends Notification {
  send(message, recipient) {
    console.log(`Sending email to ${recipient}: ${message}`);
    // Email sending logic
  }
}

class SMSNotification extends Notification {
  send(message, recipient) {
    console.log(`Sending SMS to ${recipient}: ${message}`);
    // SMS sending logic
  }
}

class PushNotification extends Notification {
  send(message, recipient) {
    console.log(`Sending push notification to ${recipient}: ${message}`);
    // Push notification logic
  }
}

// Factory - functional approach
const NotificationFactory = {
  EMAIL: () => new EmailNotification(),
  SMS: () => new SMSNotification(),
  PUSH: () => new PushNotification(),

  create(type) {
    const creator = this[type.toUpperCase()];
    if (!creator) {
      throw new Error(`Unknown notification type: ${type}`);
    }
    return creator();
  },
};

// Hoặc factory function (more JavaScript-like)
function createNotification(type) {
  const notifications = {
    email: () => new EmailNotification(),
    sms: () => new SMSNotification(),
    push: () => new PushNotification(),
  };

  const creator = notifications[type.toLowerCase()];
  if (!creator) {
    throw new Error(`Unknown notification type: ${type}`);
  }
  return creator();
}

// Sử dụng
const notification = createNotification("email");
notification.send("Hello!", "user@example.com");

// Map-based factory (modern approach)
const notificationMap = new Map([
  ["email", EmailNotification],
  ["sms", SMSNotification],
  ["push", PushNotification],
]);

function createNotificationV2(type) {
  const NotificationClass = notificationMap.get(type.toLowerCase());
  if (!NotificationClass) {
    throw new Error(`Unknown notification type: ${type}`);
  }
  return new NotificationClass();
}
```

**Nhận xét:** JavaScript có nhiều cách implement factory hơn (object literals, Maps, functions). Java structure rõ ràng hơn với enums và inheritance.

## 3. Observer Pattern - "Theo dõi và thông báo"

### Bài toán:

User management system, khi user được tạo thì cần gửi welcome email, create user profile, log audit trail.

### Java Implementation:

```java
// Observer interface
public interface UserObserver {
    void onUserCreated(User user);
    void onUserUpdated(User user);
    void onUserDeleted(String userId);
}

// Subject
public class UserService {
    private List<UserObserver> observers = new ArrayList<>();
    private UserRepository userRepository;

    public void addObserver(UserObserver observer) {
        observers.add(observer);
    }

    public void removeObserver(UserObserver observer) {
        observers.remove(observer);
    }

    private void notifyUserCreated(User user) {
        observers.forEach(observer -> observer.onUserCreated(user));
    }

    public User createUser(UserDTO userDTO) {
        User user = new User(userDTO.getName(), userDTO.getEmail());
        userRepository.save(user);

        // Notify all observers
        notifyUserCreated(user);

        return user;
    }
}

// Concrete Observers
@Component
public class EmailService implements UserObserver {
    @Override
    public void onUserCreated(User user) {
        System.out.println("Sending welcome email to: " + user.getEmail());
        // Send welcome email logic
    }

    @Override
    public void onUserUpdated(User user) {
        System.out.println("Sending update notification to: " + user.getEmail());
    }

    @Override
    public void onUserDeleted(String userId) {
        System.out.println("Sending goodbye email for user: " + userId);
    }
}

@Component
public class AuditService implements UserObserver {
    @Override
    public void onUserCreated(User user) {
        System.out.println("Logging user creation: " + user.getId());
        // Audit log logic
    }

    @Override
    public void onUserUpdated(User user) {
        System.out.println("Logging user update: " + user.getId());
    }

    @Override
    public void onUserDeleted(String userId) {
        System.out.println("Logging user deletion: " + userId);
    }
}

// Setup
UserService userService = new UserService();
userService.addObserver(new EmailService());
userService.addObserver(new AuditService());
```

### JavaScript Implementation:

```javascript
// Event-driven approach với Node.js EventEmitter
const EventEmitter = require("events");

class UserService extends EventEmitter {
  constructor(userRepository) {
    super();
    this.userRepository = userRepository;
  }

  async createUser(userData) {
    const user = {
      id: Date.now(),
      name: userData.name,
      email: userData.email,
      createdAt: new Date(),
    };

    await this.userRepository.save(user);

    // Emit event
    this.emit("userCreated", user);

    return user;
  }

  async updateUser(userId, userData) {
    const user = await this.userRepository.update(userId, userData);
    this.emit("userUpdated", user);
    return user;
  }

  async deleteUser(userId) {
    await this.userRepository.delete(userId);
    this.emit("userDeleted", userId);
  }
}

// Observers
class EmailService {
  constructor(userService) {
    userService.on("userCreated", this.onUserCreated.bind(this));
    userService.on("userUpdated", this.onUserUpdated.bind(this));
    userService.on("userDeleted", this.onUserDeleted.bind(this));
  }

  onUserCreated(user) {
    console.log(`Sending welcome email to: ${user.email}`);
    // Email logic
  }

  onUserUpdated(user) {
    console.log(`Sending update notification to: ${user.email}`);
  }

  onUserDeleted(userId) {
    console.log(`Sending goodbye email for user: ${userId}`);
  }
}

class AuditService {
  constructor(userService) {
    userService.on("userCreated", this.logUserCreation.bind(this));
    userService.on("userUpdated", this.logUserUpdate.bind(this));
    userService.on("userDeleted", this.logUserDeletion.bind(this));
  }

  logUserCreation(user) {
    console.log(`Audit: User created - ${user.id}`);
  }

  logUserUpdate(user) {
    console.log(`Audit: User updated - ${user.id}`);
  }

  logUserDeletion(userId) {
    console.log(`Audit: User deleted - ${userId}`);
  }
}

// Setup
const userService = new UserService(userRepository);
new EmailService(userService);
new AuditService(userService);

// Custom Event System (không dùng EventEmitter)
class SimpleObserver {
  constructor() {
    this.observers = new Map();
  }

  subscribe(event, callback) {
    if (!this.observers.has(event)) {
      this.observers.set(event, []);
    }
    this.observers.get(event).push(callback);
  }

  unsubscribe(event, callback) {
    if (this.observers.has(event)) {
      const callbacks = this.observers.get(event);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }

  notify(event, data) {
    if (this.observers.has(event)) {
      this.observers.get(event).forEach((callback) => callback(data));
    }
  }
}
```

## 4. Strategy Pattern - "Nhiều cách giải quyết một vấn đề"

### Bài toán:

Payment processing system với multiple payment methods (Credit Card, PayPal, Bank Transfer).

### Java Implementation:

```java
// Strategy interface
public interface PaymentStrategy {
    boolean pay(double amount, PaymentDetails details);
    String getPaymentType();
}

// Concrete strategies
public class CreditCardPayment implements PaymentStrategy {
    @Override
    public boolean pay(double amount, PaymentDetails details) {
        System.out.println("Processing credit card payment of $" + amount);
        // Credit card processing logic
        return validateCard(details.getCardNumber()) && chargeCard(amount);
    }

    @Override
    public String getPaymentType() {
        return "CREDIT_CARD";
    }

    private boolean validateCard(String cardNumber) {
        // Validation logic
        return cardNumber != null && cardNumber.length() == 16;
    }

    private boolean chargeCard(double amount) {
        // Charging logic
        return amount > 0;
    }
}

public class PayPalPayment implements PaymentStrategy {
    @Override
    public boolean pay(double amount, PaymentDetails details) {
        System.out.println("Processing PayPal payment of $" + amount);
        // PayPal API integration
        return authenticatePayPal(details.getEmail()) && processPayPal(amount);
    }

    @Override
    public String getPaymentType() {
        return "PAYPAL";
    }

    private boolean authenticatePayPal(String email) {
        return email != null && email.contains("@");
    }

    private boolean processPayPal(double amount) {
        return amount > 0;
    }
}

// Context
public class PaymentProcessor {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public boolean processPayment(double amount, PaymentDetails details) {
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment strategy not set");
        }

        System.out.println("Processing payment using " + paymentStrategy.getPaymentType());
        return paymentStrategy.pay(amount, details);
    }
}

// Usage
PaymentProcessor processor = new PaymentProcessor();

// Credit card payment
processor.setPaymentStrategy(new CreditCardPayment());
processor.processPayment(100.0, creditCardDetails);

// PayPal payment
processor.setPaymentStrategy(new PayPalPayment());
processor.processPayment(50.0, paypalDetails);
```

### JavaScript Implementation:

```javascript
// Strategy objects
const PaymentStrategies = {
  creditCard: {
    pay(amount, details) {
      console.log(`Processing credit card payment of ${amount}`);
      return this.validateCard(details.cardNumber) && this.chargeCard(amount);
    },

    validateCard(cardNumber) {
      return cardNumber && cardNumber.length === 16;
    },

    chargeCard(amount) {
      return amount > 0;
    },

    getType() {
      return "CREDIT_CARD";
    },
  },

  paypal: {
    pay(amount, details) {
      console.log(`Processing PayPal payment of ${amount}`);
      return (
        this.authenticatePayPal(details.email) && this.processPayPal(amount)
      );
    },

    authenticatePayPal(email) {
      return email && email.includes("@");
    },

    processPayPal(amount) {
      return amount > 0;
    },

    getType() {
      return "PAYPAL";
    },
  },

  bankTransfer: {
    pay(amount, details) {
      console.log(`Processing bank transfer of ${amount}`);
      return (
        this.validateAccount(details.accountNumber) &&
        this.transferFunds(amount)
      );
    },

    validateAccount(accountNumber) {
      return accountNumber && accountNumber.length >= 8;
    },

    transferFunds(amount) {
      return amount > 0;
    },

    getType() {
      return "BANK_TRANSFER";
    },
  },
};

// Context
class PaymentProcessor {
  constructor() {
    this.strategy = null;
  }

  setStrategy(strategyName) {
    const strategy = PaymentStrategies[strategyName];
    if (!strategy) {
      throw new Error(`Unknown payment strategy: ${strategyName}`);
    }
    this.strategy = strategy;
    return this;
  }

  processPayment(amount, details) {
    if (!this.strategy) {
      throw new Error("Payment strategy not set");
    }

    console.log(`Processing payment using ${this.strategy.getType()}`);
    return this.strategy.pay(amount, details);
  }
}

// Functional approach
function createPaymentProcessor(strategies) {
  let currentStrategy = null;

  return {
    setStrategy(strategyName) {
      if (!strategies[strategyName]) {
        throw new Error(`Unknown payment strategy: ${strategyName}`);
      }
      currentStrategy = strategies[strategyName];
      return this;
    },

    processPayment(amount, details) {
      if (!currentStrategy) {
        throw new Error("Payment strategy not set");
      }
      return currentStrategy(amount, details);
    },
  };
}

// Functional strategies
const functionalStrategies = {
  creditCard: (amount, details) => {
    console.log(`Credit card payment: ${amount}`);
    return details.cardNumber && details.cardNumber.length === 16;
  },

  paypal: (amount, details) => {
    console.log(`PayPal payment: ${amount}`);
    return details.email && details.email.includes("@");
  },
};

// Usage
const processor = new PaymentProcessor();

// Credit card
processor
  .setStrategy("creditCard")
  .processPayment(100, { cardNumber: "1234567890123456" });

// PayPal
processor
  .setStrategy("paypal")
  .processPayment(50, { email: "user@example.com" });

// Functional approach
const functionalProcessor = createPaymentProcessor(functionalStrategies);
functionalProcessor
  .setStrategy("creditCard")
  .processPayment(75, { cardNumber: "1234567890123456" });
```

## So sánh và Lessons Learned

### 1. **Complexity vs Flexibility**

- **Java**: Structure rõ ràng, type-safe, nhưng verbose hơn
- **JavaScript**: Flexible, concise, nhưng dễ messy nếu không cẩn thận

### 2. **Performance**

- **Java**: Compile-time optimization, JVM tuning
- **JavaScript**: V8 engine optimization, nhưng dynamic typing overhead

### 3. **Team Scalability**

- **Java**: Dễ onboard developers mới nhờ structure rõ ràng
- **JavaScript**: Cần conventions và discipline cao hơn

### 4. **When to use what?**

**Java Design Patterns khi:**

- Large team, long-term maintenance
- Complex business logic
- Enterprise applications
- Type safety quan trọng

**JavaScript Design Patterns khi:**

- Rapid prototyping
- Small to medium team
- Web-first applications
- Flexibility quan trọng hơn structure

## Kết luận

Design patterns không phải là silver bullet, mà là tools giúp solve common problems. Key takeaways:

1. **Hiểu problem trước khi apply pattern**
2. **JavaScript có thể implement patterns theo nhiều cách**
3. **Java patterns thường require more boilerplate**
4. **Choose based on team, project, và requirements**

Các bạn đã apply pattern nào trong dự án chưa? Share experience ở comment nhé!

---

_Patterns are guides, not rules! 🎯📐_
