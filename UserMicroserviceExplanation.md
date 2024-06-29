1. Application Class:

Earlier (Monolith):

```java
@SpringBootApplication
public class BlogappApplication {
    public static void main(String[] args) {
        SpringApplication.run(BlogappApplication.class, args);
    }
}
```

Current (Microservice):

```java
@SpringBootApplication
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

The main change here is the class name, reflecting its specific purpose as a user service.

2. User Model:

The User model remains largely the same, but it's now part of the user-specific package:

```java
package com.example.userservice.model;

@Entity
@Table(name = "users")
public class User {
    // Fields and methods remain the same
}
```

3. UserRepository:

The repository interface remains similar, but it's now in a user-specific package:

```java
package com.example.userservice.repository;

public interface UserRepository extends JpaRepository<User, Integer> {
}
```

4. UserService:

Earlier (Monolith):

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    // Only had create user method
}
```

Current (Microservice):

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public User getUserById(int userId) {
        return userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found with id: " + userId));
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public User updateUser(int userId, User userDetails) {
        User user = getUserById(userId);
        user.setUsername(userDetails.getUsername());
        user.setEmail(userDetails.getEmail());
        return userRepository.save(user);
    }

    public void deleteUser(int userId) {
        User user = getUserById(userId);
        userRepository.delete(user);
    }
}
```

The microservice version has more comprehensive CRUD operations.

5. UserController:

Earlier (Monolith):

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }

    // Only had create user endpoint
}
```

Current (Microservice):

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        return ResponseEntity.ok(userService.createUser(user));
    }

    @GetMapping("/{userId}")
    public ResponseEntity<User> getUserById(@PathVariable int userId) {
        return ResponseEntity.ok(userService.getUserById(userId));
    }

    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }

    @PutMapping("/{userId}")
    public ResponseEntity<User> updateUser(@PathVariable int userId, @RequestBody User userDetails) {
        return ResponseEntity.ok(userService.updateUser(userId, userDetails));
    }

    @DeleteMapping("/{userId}")
    public ResponseEntity<?> deleteUser(@PathVariable int userId) {
        userService.deleteUser(userId);
        return ResponseEntity.ok().build();
    }
}
```

The microservice controller now offers a full set of CRUD operations and uses ResponseEntity for more flexible HTTP responses.

Key Changes:

1. Separation of concerns: The user-related code is now in its own microservice, independent of other functionalities.
2. Expanded functionality: The microservice offers more comprehensive user management operations.
3. Package structure: All user-related classes are now under a user-specific package hierarchy.
4. Independence: The UserServiceApplication can run as a standalone service.
5. Scalability: This separation allows the user service to be scaled independently of other services.

These changes allow the user functionality to operate as an independent service, communicating with other services (like the post service) via REST APIs, thus adhering to microservice architecture principles.
