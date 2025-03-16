### Lombok Annotations Overview

Lombok is a Java library that helps reduce boilerplate code by generating common methods like getters, setters, constructors, etc., through annotations.

Here's a list of essential Lombok annotations and examples of their usage:

#### 1. **@Data**

- Combines several other Lombok annotations to generate getters, setters, `equals()`, `hashCode()`, `toString()`, and a required arguments constructor.

**Without @Data:**

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        // equals implementation
    }

    @Override
    public int hashCode() {
        // hashCode implementation
    }

    @Override
    public String toString() {
        return "User{name=" + name + ", age=" + age + "}";
    }
}
```

**With @Data:**

```java
import lombok.Data;

@Data
public class User {
    private String name;
    private int age;
}
```

#### 2. **@Builder**

- Provides a builder pattern for object creation.

**Without @Builder:**

```java
public class User {
    private String name;
    private int age;

    private User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static class UserBuilder {
        private String name;
        private int age;

        public UserBuilder name(String name) {
            this.name = name;
            return this;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public User build() {
            return new User(name, age);
        }
    }

    public static UserBuilder builder() {
        return new UserBuilder();
    }
}
```

**With @Builder:**

```java
import lombok.Builder;

@Builder
public class User {
    private String name;
    private int age;
}

// Usage:
User user = User.builder().name("John").age(30).build();
```

#### 3. **@AllArgsConstructor**

- Generates a constructor with all fields as parameters.

**Without @AllArgsConstructor:**

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

**With @AllArgsConstructor:**

```java
import lombok.AllArgsConstructor;

@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```

#### 4. **@NoArgsConstructor**

- Generates a no-arguments constructor.

**Without @NoArgsConstructor:**

```java
public class User {
    private String name;
    private int age;

    public User() {
    }
}
```

**With @NoArgsConstructor:**

```java
import lombok.NoArgsConstructor;

@NoArgsConstructor
public class User {
    private String name;
    private int age;
}
```

#### 5. **@Getter and @Setter**

- Generates getter and setter methods for each field.

**Without @Getter and @Setter:**

```java
public class User {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

**With @Getter and @Setter:**

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User {
    private String name;
    private int age;
}
```

#### 6. **@ToString**

- Generates a `toString()` method.

**Without @ToString:**

```java
public class User {
    private String name;
    private int age;

    @Override
    public String toString() {
        return "User{name=" + name + ", age=" + age + "}";
    }
}
```

**With @ToString:**

```java
import lombok.ToString;

@ToString
public class User {
    private String name;
    private int age;
}
```

### Practice Exercise

- **Task:** Implement a `Product` class using Lombok annotations to generate a builder, toString, getters, setters, and a constructor with all arguments.
  
- **Steps:**
  1. Create a `Product` class with fields like `id`, `name`, and `price`.
  2. Use `@Data` for basic methods.
  3. Use `@Builder` for the builder pattern.
  4. Write a main method to create and display a `Product` instance. 

By using Lombok, you can significantly reduce boilerplate code and make your classes cleaner and more readable.
