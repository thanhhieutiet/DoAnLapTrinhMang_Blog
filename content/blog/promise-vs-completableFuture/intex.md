---
title: "Async Programming: Promise (JS) vs CompletableFuture (Java) - Battle of Asynchronous!"
date: 2024-09-28
tags: ["javascript", "java", "async", "promise", "completablefuture"]
categories: ["Async Programming"]
author: "Tên bạn"
description: "So sánh cách xử lý bất đồng bộ giữa JavaScript Promise và Java CompletableFuture"
featuredImage: "/blog/java-socket/featured.jpg"
---

Chào các bạn! Async programming là một trong những concepts khó nhất khi học lập trình. Mình đã struggle với nó khá lâu, từ callback hell của JavaScript đến thread complexity của Java. Hôm nay mình muốn chia sẻ cách hai ngôn ngữ này handle async operations - JavaScript với Promise và Java với CompletableFuture.

## Câu chuyện bắt đầu từ Synchronous Hell

### JavaScript - Callback Hell

```javascript
// Cách xưa - callback nightmare!
function getUserData(userId, callback) {
  fetchUser(userId, (user) => {
    if (user) {
      fetchUserProfile(user.id, (profile) => {
        if (profile) {
          fetchUserPosts(user.id, (posts) => {
            if (posts) {
              fetchPostComments(posts[0].id, (comments) => {
                // 4 levels deep! Pyramid of doom 😱
                callback({ user, profile, posts, comments });
              });
            } else {
              callback(null, "No posts found");
            }
          });
        } else {
          callback(null, "Profile not found");
        }
      });
    } else {
      callback(null, "User not found");
    }
  });
}
```

### Java - Thread Complexity

```java
// Cách xưa với Thread
public void getUserDataOldWay(String userId) {
    new Thread(() -> {
        try {
            User user = fetchUser(userId);
            if (user != null) {
                new Thread(() -> {
                    try {
                        Profile profile = fetchUserProfile(user.getId());
                        if (profile != null) {
                            // Nested threads everywhere! 😫
                            // Error handling nightmare
                            // No easy way to combine results
                        }
                    } catch (Exception e) {
                        // Handle error
                    }
                }).start();
            }
        } catch (Exception e) {
            // Handle error
        }
    }).start();
}
```

## JavaScript Promise - Giải cứu từ Callback Hell

### Promise Basics

```javascript
// Tạo Promise đơn giản
function fetchUser(userId) {
  return new Promise((resolve, reject) => {
    // Simulate API call
    setTimeout(() => {
      if (userId > 0) {
        resolve({ id: userId, name: `User ${userId}` });
      } else {
        reject(new Error("Invalid user ID"));
      }
    }, 1000);
  });
}

// Sử dụng Promise
fetchUser(123)
  .then((user) => {
    console.log("User:", user);
    return user; // Pass data to next .then()
  })
  .then((user) => {
    // Chain another async operation
    return fetchUserProfile(user.id);
  })
  .then((profile) => {
    console.log("Profile:", profile);
  })
  .catch((error) => {
    console.error("Error:", error);
  })
  .finally(() => {
    console.log("Cleanup code here");
  });
```

### Promise.all() - Chạy song song

```javascript
// Thay vì chạy tuần tự
async function getUserDataSequential(userId) {
  const user = await fetchUser(userId); // 1s
  const profile = await fetchUserProfile(userId); // 1s
  const posts = await fetchUserPosts(userId); // 1s
  // Total: 3s

  return { user, profile, posts };
}

// Chạy song song với Promise.all()
async function getUserDataParallel(userId) {
  const [user, profile, posts] = await Promise.all([
    fetchUser(userId), // All run in parallel
    fetchUserProfile(userId), // Total: 1s (longest operation)
    fetchUserPosts(userId),
  ]);

  return { user, profile, posts };
}

// Promise.allSettled() - Không fail nếu 1 promise reject
async function getUserDataSafe(userId) {
  const results = await Promise.allSettled([
    fetchUser(userId),
    fetchUserProfile(userId),
    fetchUserPosts(userId),
  ]);

  return results.map((result) => {
    if (result.status === "fulfilled") {
      return result.value;
    } else {
      console.error("Failed:", result.reason);
      return null;
    }
  });
}
```

### Async/Await - Promise syntax đường

```javascript
// Từ Promise chain
function getUserData(userId) {
  return fetchUser(userId)
    .then((user) => fetchUserProfile(user.id))
    .then((profile) => fetchUserPosts(profile.userId))
    .then((posts) => ({ user, profile, posts }))
    .catch((error) => {
      console.error("Error:", error);
      throw error;
    });
}

// Đến async/await - clean hơn nhiều!
async function getUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const profile = await fetchUserProfile(user.id);
    const posts = await fetchUserPosts(profile.userId);

    return { user, profile, posts };
  } catch (error) {
    console.error("Error:", error);
    throw error;
  }
}
```

## Java CompletableFuture - Modern Async Java

### CompletableFuture Basics

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class AsyncService {
    private ExecutorService executor = Executors.newFixedThreadPool(10);

    // Tạo CompletableFuture đơn giản
    public CompletableFuture<User> fetchUser(String userId) {
        return CompletableFuture.supplyAsync(() -> {
            // Simulate delay
            try {
                Thread.sleep(1000);
                if (Integer.parseInt(userId) > 0) {
                    return new User(userId, "User " + userId);
                } else {
                    throw new RuntimeException("Invalid user ID");
                }
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }, executor);
    }

    // Chain operations
    public CompletableFuture<UserData> getUserData(String userId) {
        return fetchUser(userId)
            .thenCompose(user ->
                fetchUserProfile(user.getId())
                    .thenApply(profile -> new UserData(user, profile))
            )
            .thenCompose(userData ->
                fetchUserPosts(userData.getUser().getId())
                    .thenApply(posts -> {
                        userData.setPosts(posts);
                        return userData;
                    })
            )
            .exceptionally(throwable -> {
                System.err.println("Error: " + throwable.getMessage());
                return null;
            });
    }
}
```

### Parallel Execution với CompletableFuture

```java
public CompletableFuture<UserData> getUserDataParallel(String userId) {
    CompletableFuture<User> userFuture = fetchUser(userId);
    CompletableFuture<Profile> profileFuture = fetchUserProfile(userId);
    CompletableFuture<List<Post>> postsFuture = fetchUserPosts(userId);

    // Combine all results
    return userFuture.thenCombine(profileFuture, (user, profile) ->
        new UserData(user, profile)
    ).thenCombine(postsFuture, (userData, posts) -> {
        userData.setPosts(posts);
        return userData;
    });
}

// allOf() - giống Promise.all()
public CompletableFuture<List<String>> fetchMultipleUsers(List<String> userIds) {
    List<CompletableFuture<User>> futures = userIds.stream()
        .map(this::fetchUser)
        .collect(Collectors.toList());

    CompletableFuture<Void> allFutures = CompletableFuture.allOf(
        futures.toArray(new CompletableFuture[0])
    );

    return allFutures.thenApply(v ->
        futures.stream()
            .map(CompletableFuture::join) // Get results
            .map(User::getName)
            .collect(Collectors.toList())
    );
}
```

## So sánh trực tiếp Promise vs CompletableFuture

### 1. Tạo async operation

**JavaScript:**

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("Done!"), 1000);
});

// Hoặc với async function
const asyncFunction = async () => {
  return "Done!";
};
```

**Java:**

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try { Thread.sleep(1000); } catch (InterruptedException e) {}
    return "Done!";
});

// Hoặc completed future
CompletableFuture<String> completedFuture = CompletableFuture.completedFuture("Done!");
```

### 2. Chain operations

**JavaScript:**

```javascript
promise
  .then((result) => result.toUpperCase())
  .then((upper) => upper + " - FINISHED")
  .catch((error) => "Error: " + error);
```

**Java:**

```java
future
    .thenApply(result -> result.toUpperCase())
    .thenApply(upper -> upper + " - FINISHED")
    .exceptionally(error -> "Error: " + error.getMessage());
```

### 3. Combine multiple operations

**JavaScript:**

```javascript
Promise.all([promise1, promise2, promise3]).then(
  ([result1, result2, result3]) => {
    return { result1, result2, result3 };
  }
);
```

**Java:**

```java
future1.thenCombine(future2, (r1, r2) -> new Pair(r1, r2))
    .thenCombine(future3, (pair, r3) ->
        new Result(pair.getFirst(), pair.getSecond(), r3)
    );

// Hoặc với allOf
CompletableFuture.allOf(future1, future2, future3)
    .thenApply(v -> new Result(
        future1.join(),
        future2.join(),
        future3.join()
    ));
```

## Real-world Example: API Gateway

### JavaScript Implementation:

```javascript
class APIGateway {
  async handleRequest(request) {
    try {
      // Authenticate user
      const user = await this.authenticate(request.token);

      // Fetch data in parallel
      const [userData, permissions, preferences] = await Promise.all([
        this.fetchUserData(user.id),
        this.fetchUserPermissions(user.id),
        this.fetchUserPreferences(user.id),
      ]);

      // Transform data
      const response = await this.transformResponse({
        user: userData,
        permissions,
        preferences,
      });

      return response;
    } catch (error) {
      return this.handleError(error);
    }
  }

  async fetchUserData(userId) {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  }
}
```

### Java Implementation:

```java
@Service
public class APIGateway {

    private final ExecutorService executor = Executors.newFixedThreadPool(20);

    public CompletableFuture<APIResponse> handleRequest(APIRequest request) {
        return authenticate(request.getToken())
            .thenCompose(user -> {
                // Fetch data in parallel
                CompletableFuture<UserData> userDataFuture =
                    fetchUserData(user.getId());
                CompletableFuture<List<Permission>> permissionsFuture =
                    fetchUserPermissions(user.getId());
                CompletableFuture<Preferences> preferencesFuture =
                    fetchUserPreferences(user.getId());

                return userDataFuture.thenCombine(permissionsFuture,
                        (userData, permissions) -> new Pair<>(userData, permissions))
                    .thenCombine(preferencesFuture, (pair, preferences) ->
                        new RequestData(pair.getFirst(), pair.getSecond(), preferences)
                    );
            })
            .thenCompose(this::transformResponse)
            .exceptionally(this::handleError);
    }

    private CompletableFuture<UserData> fetchUserData(String userId) {
        return CompletableFuture.supplyAsync(() -> {
            // HTTP call implementation
            try {
                return restTemplate.getForObject("/api/users/" + userId, UserData.class);
            } catch (Exception e) {
                throw new RuntimeException("Failed to fetch user data", e);
            }
        }, executor);
    }
}
```

## Performance & Lessons Learned

### Performance Comparison:

- **JavaScript Promise**: Single-threaded event loop, excellent cho I/O operations
- **Java CompletableFuture**: Multi-threaded, tốt cho CPU-intensive tasks

### Best Practices:

**JavaScript:**

1. Luôn dùng `async/await` thay vì `.then()` chains
2. Dùng `Promise.all()` cho parallel operations
3. Handle errors với try/catch
4. Avoid creating unnecessary Promises

**Java:**

1. Sử dụng custom ExecutorService
2. Handle exceptions với `exceptionally()` hoặc `handle()`
3. Dùng `thenCombine()` cho parallel operations
4. Remember to shutdown ExecutorService

## Kết luận

Cả Promise và CompletableFuture đều là tools mạnh mẽ cho async programming:

- **Promise**: Đơn giản hơn, syntax clean với async/await
- **CompletableFuture**: Mạnh mẽ hơn, control tốt hơn threading

Quan trọng là hiểu concept async programming, sau đó syntax chỉ là matter of practice!

Các bạn đã dùng async programming chưa? Share kinh nghiệm ở comment nhé!

---

_Async is the future! ⚡🚀_
