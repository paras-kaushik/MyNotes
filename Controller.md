Spring MVC is a powerful framework that allows you to create web applications with various types of controllers. In this masterclass, we'll explore different ways to create controllers in Spring MVC, focusing on RESTful and traditional web controllers. We'll also delve into the differences between `@RequestParam` and `@RequestBody`, and demonstrate how to handle different HTTP methods and parameter types.

### 1. Types of Controllers in Spring MVC

#### Traditional Web Controller

A traditional web controller in Spring MVC is used to handle web requests and return views (like JSP, Thymeleaf).

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
@RequestMapping("/web")
public class WebController {

    @GetMapping("/greeting")
    public ModelAndView greeting() {
        ModelAndView modelAndView = new ModelAndView("greeting");
        modelAndView.addObject("message", "Hello, World!");
        return modelAndView; // Returns a view named 'greeting'
    }
}
```

**URL**: `http://localhost:8080/web/greeting`

#### RESTful Controller

A RESTful controller is used to handle REST API requests and typically returns data in JSON or XML format.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class RestApiController {

    @GetMapping("/greeting")
    public String greeting() {
        return "Hello, World!"; // Returns a plain text response
    }
}
```

**URL**: `http://localhost:8080/api/greeting`

### 2. @RequestParam vs @RequestBody

- **@RequestParam**: Used to extract query parameters from the URL.
- **@RequestBody**: Used to bind the HTTP request body to a method parameter.

#### Example of @RequestParam

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class RequestParamController {

    @GetMapping("/welcome")
    public String welcome(@RequestParam String name) {
        return "Welcome, " + name + "!";
    }
}
```

**URL**: `http://localhost:8080/api/welcome?name=John`

#### Example of @RequestBody

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class RequestBodyController {

    @PostMapping("/echo")
    public String echo(@RequestBody String message) {
        return "Echo: " + message;
    }
}
```

**URL**: `http://localhost:8080/api/echo` (POST request with body: "Hello")

### 3. Handling Different HTTP Methods

#### GET Method

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class GetController {

    @GetMapping("/user/{id}")
    public String getUser(@PathVariable int id) {
        return "User ID: " + id;
    }
}
```

**URL**: `http://localhost:8080/api/user/123`

#### POST Method

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class PostController {

    @PostMapping("/user")
    public String createUser(@RequestBody String user) {
        return "Created User: " + user;
    }
}
```

**URL**: `http://localhost:8080/api/user` (POST request with body: JSON user data)

#### PUT Method

```java
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class PutController {

    @PutMapping("/user")
    public String updateUser(@RequestBody String user) {
        return "Updated User: " + user;
    }
}
```

**URL**: `http://localhost:8080/api/user` (PUT request with body: JSON user data)

#### DELETE Method

```java
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class DeleteController {

    @DeleteMapping("/user/{id}")
    public String deleteUser(@PathVariable int id) {
        return "Deleted User ID: " + id;
    }
}
```

**URL**: `http://localhost:8080/api/user/123`

#### PATCH Method

```java
import org.springframework.web.bind.annotation.PatchMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class PatchController {

    @PatchMapping("/user")
    public String patchUser(@RequestBody String user) {
        return "Patched User: " + user;
    }
}
```

**URL**: `http://localhost:8080/api/user` (PATCH request with body: JSON user data)

### 4. Handling Path Params vs Query Params vs Request Body

- **Path Params**: Use `@PathVariable` to extract values from the URI path.
- **Query Params**: Use `@RequestParam` to extract values from the query string.
- **Request Body**: Use `@RequestBody` to bind the request body to a method parameter.

#### Example with All Three

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class MixedParamsController {

    @GetMapping("/product/{id}")
    public String getProduct(
            @PathVariable int id,
            @RequestParam String name,
            @RequestBody(required = false) String description) {
        return "Product ID: " + id + ", Name: " + name + ", Description: " + description;
    }
}
```

**URL**: `http://localhost:8080/api/product/123?name=Widget` (GET request with optional body: "A great widget")

### Conclusion

Spring MVC provides a flexible way to create both traditional web and RESTful controllers. Understanding the differences between `@RequestParam` and `@RequestBody`, as well as how to handle various HTTP methods and parameter types, is crucial for building robust web applications. For further reading, consider exploring the [Spring Framework Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html) for more in-depth information.
