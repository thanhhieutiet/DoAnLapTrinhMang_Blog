---
title: "JavaScript ES6+: Những tính năng đã thay đổi cách mình code"
date: 2024-01-22T09:15:00+07:00
tags: ["javascript", "es6", "modern-js", "kinh-nghiem"]
categories: ["JavaScript"]
author: "Tên bạn"
description: "Khám phá những tính năng ES6+ đã cách mạng hóa cách viết JavaScript của mình"
featuredImage: "/images/featured.jpg"
---

Chào mọi người! Khi mới học JavaScript, mình chỉ biết đến cú pháp cũ ES5. Nhưng khi tiếp xúc với ES6+ (ES2015 trở lên), thật sự mình cảm thấy như được "khai sáng". Hôm nay mình muốn chia sẻ những tính năng ES6+ đã thay đổi hoàn toàn cách mình viết JavaScript.

## 1. Let & Const - Tạm biệt `var` nhé!

### Cách cũ với `var`:

```javascript
// Vấn đề với var
function oldWay() {
  for (var i = 0; i < 3; i++) {
    setTimeout(() => {
      console.log(i); // In ra 3, 3, 3 (WTF???)
    }, 100);
  }
  console.log(i); // 3 - i vẫn tồn tại ngoài loop!
}
```

### Cách mới với `let` & `const`:

```javascript
function newWay() {
  for (let i = 0; i < 3; i++) {
    setTimeout(() => {
      console.log(i); // In ra 0, 1, 2 (chính xác!)
    }, 100);
  }
  // console.log(i); // ReferenceError - i không tồn tại ở đây

  const API_URL = "https://api.example.com";
  // API_URL = 'other-url'; // TypeError - không thể reassign const
}
```

**Bài học:** Luôn dùng `const` cho giá trị không thay đổi, `let` cho biến có thể thay đổi. Quên `var` đi!

## 2. Arrow Functions - Code ngắn gọn hơn

### ES5 way:

```javascript
// Function expression cũ
var users = ["Nam", "Linh", "Hoa"];

var uppercaseUsers = users.map(function (user) {
  return user.toUpperCase();
});

var button = document.getElementById("btn");
button.addEventListener("click", function () {
  console.log(this); // 'this' trỏ đến button
});
```

### ES6 way:

```javascript
// Arrow function - clean và concise
const users = ["Nam", "Linh", "Hoa"];

const uppercaseUsers = users.map((user) => user.toUpperCase());

// Với multiple parameters
const add = (a, b) => a + b;

// Với function body
const processUser = (user) => {
  const processed = user.trim().toUpperCase();
  return `Hello, ${processed}!`;
};

// Lưu ý về 'this' binding
const obj = {
  name: "MyObject",
  regularFunction() {
    console.log(this.name); // 'MyObject'

    const arrowFunction = () => {
      console.log(this.name); // 'MyObject' - inherit từ parent scope
    };
    arrowFunction();
  },
};
```

**Tip:** Arrow function không có `this` binding riêng, nó inherit từ parent scope.

## 3. Template Literals - Goodbye string concatenation!

### Cách cũ:

```javascript
var name = "Nam";
var age = 20;
var city = "Ho Chi Minh";

var introduction =
  "Xin chào, tôi là " + name + ", " + age + " tuổi, " + "đến từ " + city + ".";

var htmlTemplate =
  '<div class="user">' +
  "<h2>" +
  name +
  "</h2>" +
  "<p>Age: " +
  age +
  "</p>" +
  "</div>";
```

### Cách mới:

```javascript
const name = "Nam";
const age = 20;
const city = "Ho Chi Minh";

// Template literals với backticks
const introduction = `Xin chào, tôi là ${name}, ${age} tuổi, đến từ ${city}.`;

// Multi-line strings dễ dàng
const htmlTemplate = `
    <div class="user">
        <h2>${name}</h2>
        <p>Age: ${age}</p>
        <p>Status: ${age >= 18 ? "Adult" : "Minor"}</p>
    </div>
`;

// Tagged template literals (advanced)
function highlight(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] ? `<mark>${values[i]}</mark>` : "";
    return result + string + value;
  }, "");
}

const highlighted = highlight`Tôi tên là ${name} và ${age} tuổi.`;
```

## 4. Destructuring - Unpack dữ liệu như ninja

### Array Destructuring:

```javascript
// Cách cũ
var numbers = [1, 2, 3, 4, 5];
var first = numbers[0];
var second = numbers[1];
var rest = numbers.slice(2);

// Cách mới
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(second); // 2
console.log(rest); // [3, 4, 5]

// Skip elements
const [a, , c] = [1, 2, 3];
console.log(a, c); // 1, 3

// Default values
const [x = 0, y = 0] = [1]; // x = 1, y = 0
```

### Object Destructuring:

```javascript
// Cách cũ
var user = {
  name: "Nam",
  age: 20,
  email: "nam@example.com",
  address: {
    city: "Ho Chi Minh",
    district: "District 1",
  },
};

var name = user.name;
var age = user.age;
var email = user.email;

// Cách mới
const { name, age, email } = user;

// Rename variables
const { name: userName, age: userAge } = user;

// Nested destructuring
const {
  address: { city, district },
} = user;

// Default values
const { phone = "N/A" } = user;

// Function parameters destructuring
function greetUser({ name, age = "unknown" }) {
  return `Hello ${name}, you are ${age} years old`;
}

greetUser({ name: "Nam", age: 20 });
```

## 5. Spread & Rest Operators - Ba chấm ma thuật

### Spread Operator (...):

```javascript
// Array spreading
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Object spreading
const user = { name: "Nam", age: 20 };
const updatedUser = { ...user, age: 21, city: "HCM" };
// { name: 'Nam', age: 21, city: 'HCM' }

// Function arguments
function sum(a, b, c) {
  return a + b + c;
}
const numbers = [1, 2, 3];
console.log(sum(...numbers)); // 6
```

### Rest Operator (...):

```javascript
// Function parameters
function logAll(first, ...others) {
  console.log("First:", first);
  console.log("Others:", others);
}
logAll(1, 2, 3, 4, 5); // First: 1, Others: [2, 3, 4, 5]

// Array destructuring (đã thấy ở trên)
const [head, ...tail] = [1, 2, 3, 4, 5];
```

## 6. Enhanced Object Literals

```javascript
const name = "Nam";
const age = 20;

// Cách cũ
var user = {
  name: name,
  age: age,
  greet: function () {
    return "Hello!";
  },
};

// Cách mới - shorthand properties
const user = {
  name, // tương đương name: name
  age, // tương đương age: age

  // Method shorthand
  greet() {
    return "Hello!";
  },

  // Computed property names
  [`full${name}`]: `${name} Nguyen`,

  // Dynamic property
  [getPropertyName()]: "dynamic value",
};
```

## 7. Promises & Async/Await - Tạm biệt Callback Hell

### Callback Hell:

```javascript
// Cách cũ - callback hell
fetchUser(userId, function (user) {
  fetchUserPosts(user.id, function (posts) {
    fetchPostComments(posts[0].id, function (comments) {
      // Nested callbacks - khó đọc và maintain
      console.log(comments);
    });
  });
});
```

### Promises:

```javascript
// Promise chain
fetchUser(userId)
  .then((user) => fetchUserPosts(user.id))
  .then((posts) => fetchPostComments(posts[0].id))
  .then((comments) => console.log(comments))
  .catch((error) => console.error(error));
```

### Async/Await:

```javascript
// Async/await - code như synchronous
async function getUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchUserPosts(user.id);
    const comments = await fetchPostComments(posts[0].id);
    return comments;
  } catch (error) {
    console.error("Error:", error);
    throw error;
  }
}

// Sử dụng
getUserData(123).then((data) => console.log(data));
```

## Kết luận

ES6+ đã thực sự thay đổi cách mình viết JavaScript:

- Code ngắn gọn và dễ đọc hơn
- Ít bugs hơn nhờ `let/const`
- Xử lý async code dễ dàng hơn
- Destructuring giúp unpack data elegantly

### Tips cho người mới:

1. **Học từ từ** - Đừng cố gắng học tất cả một lúc
2. **Practice daily** - Viết code ES6+ mỗi ngày
3. **Use Babel** - Để support browser cũ
4. **Read MDN docs** - Tài liệu chính thức luôn tốt nhất

Các bạn đã dùng feature nào trong số này chưa? Feature nào khiến các bạn ấn tượng nhất? Share ở comment nhé!

---

_Modern JavaScript is beautiful! 🚀✨_
