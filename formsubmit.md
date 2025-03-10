Certainly! Let's create a detailed example of a Spring Boot application with a Thymeleaf form and a corresponding POST controller. This example will demonstrate a simple user registration form where users can enter their name, email, and password. The form data will be submitted to a POST controller, which will process the data and return a confirmation view.

### Project Structure

```
spring-boot-thymeleaf-form
│
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com.example.demo
│   │   │       ├── DemoApplication.java
│   │   │       ├── controller
│   │   │       │   └── UserController.java
│   │   │       └── model
│   │   │           └── User.java
│   │   └── resources
│   │       ├── templates
│   │       │   ├── register.html
│   │       │   └── result.html
│   │       └── application.properties
└── pom.xml
```

### Step 1: Set Up the Spring Boot Application

**pom.xml**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>spring-boot-thymeleaf-form</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Step 2: Create the Model

**User.java**

```java
package com.example.demo.model;

public class User {
    private String name;
    private String email;
    private String password;

    // Getters and setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### Step 3: Create the Controller

**UserController.java**

```java
package com.example.demo.controller;

import com.example.demo.model.User;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class UserController {

    @GetMapping("/register")
    public String showRegistrationForm(Model model) {
        model.addAttribute("user", new User());
        return "register";
    }

    @PostMapping("/register")
    public String registerUser(@ModelAttribute User user, Model model) {
        // Process the user data (e.g., save to database)
        // For this example, we'll just print the user data to the console
        System.out.println("User registered: " + user.getName() + ", " + user.getEmail());

        // Add the user to the model to display in the result view
        model.addAttribute("user", user);
        return "result";
    }
}
```

### Step 4: Create the Thymeleaf Templates

**register.html**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>User Registration</title>
</head>
<body>
    <h1>Register</h1>
    <form th:action="@{/register}" th:object="${user}" method="post">
        <div>
            <label for="name">Name:</label>
            <input type="text" id="name" th:field="*{name}" required />
        </div>
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" th:field="*{email}" required />
        </div>
        <div>
            <label for="password">Password:</label>
            <input type="password" id="password" th:field="*{password}" required />
        </div>
        <div>
            <button type="submit">Register</button>
        </div>
    </form>
</body>
</html>
```

**result.html**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Registration Result</title>
</head>
<body>
    <h1>Registration Successful</h1>
    <p>Thank you for registering, <span th:text="${user.name}"></span>!</p>
    <p>Your email: <span th:text="${user.email}"></span></p>
</body>
</html>
```

### Step 5: Main Application Class

**DemoApplication.java**

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### Explanation

- **Model**: The `User` class represents the data structure for the form.
- **Controller**: The `UserController` handles GET requests to display the registration form and POST requests to process the form submission.
- **View**: The `register.html` file is the form where users enter their details. The `result.html` file displays a confirmation message after successful registration.
- **Thymeleaf**: The `th:action` attribute specifies the form's action URL, and `th:field` binds form fields to the `User` object.

### Running the Application

1. Ensure you have Maven and Java installed.
2. Navigate to the project directory and run `mvn spring-boot:run`.
3. Open a web browser and go to `http://localhost:8080/register` to see the registration form.
4. Fill out the form and submit it to see the result page.

This example demonstrates a simple user registration form using Thymeleaf and Spring Boot, showcasing how to handle form submissions and display results. You can expand this example by adding validation, error handling, and database integration.
