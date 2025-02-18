Certainly! Let's start by understanding a basic servlet-based Todo application and then convert it into a Spring Boot REST API.

### Servlet-Based Todo Application

**Explanation**:
A servlet-based application typically involves writing servlets that handle HTTP requests and responses. For a Todo app, you might have servlets to add, list, update, and delete todo items.

**Servlet Code Example**:

1. **TodoServlet.java**: A simple servlet to handle HTTP requests for managing todos.

```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

@WebServlet("/todos")
public class TodoServlet extends HttpServlet {

    private List<String> todos = new ArrayList<>();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        resp.getWriter().println("<h1>Todo List</h1>");
        for (String todo : todos) {
            resp.getWriter().println("<p>" + todo + "</p>");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String todo = req.getParameter("todo");
        if (todo != null && !todo.trim().isEmpty()) {
            todos.add(todo);
        }
        resp.sendRedirect("/todos");
    }
}
```

**Explanation**:
- **doGet**: Handles GET requests to list all todos.
- **doPost**: Handles POST requests to add a new todo.

### Converting to Spring Boot REST API

**Step 1: Set Up Spring Boot Project**

1. **Create a new Spring Boot project** using Spring Initializr (https://start.spring.io/).
   - Choose dependencies: Spring Web, Spring Boot DevTools (optional for development).

**Step 2: Define the Todo Entity**

2. **Create a Todo class** to represent a todo item.

```java
public class Todo {
    private Long id;
    private String description;

    // Constructors, getters, and setters
    public Todo() {}

    public Todo(Long id, String description) {
        this.id = id;
        this.description = description;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

**Step 3: Create a Controller**

3. **Create a TodoController** to handle HTTP requests.

```java
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@RestController
@RequestMapping("/api/todos")
public class TodoController {

    private List<Todo> todos = new ArrayList<>();
    private AtomicLong counter = new AtomicLong();

    @GetMapping
    public List<Todo> getAllTodos() {
        return todos; // Return the list of todos
    }

    @PostMapping
    public Todo addTodo(@RequestBody Todo todo) {
        todo.setId(counter.incrementAndGet()); // Set a unique ID
        todos.add(todo); // Add the new todo to the list
        return todo; // Return the added todo
    }
}
```

**Explanation**:
- **@RestController**: Indicates that this class is a REST controller.
- **@RequestMapping("/api/todos")**: Maps requests to `/api/todos`.
- **@GetMapping**: Handles GET requests to retrieve all todos.
- **@PostMapping**: Handles POST requests to add a new todo.

**Step 4: Run the Application**

4. **Run the Spring Boot application**.
   - Use `mvn spring-boot:run` or run the main class from your IDE.

**Step 5: Test the API**

5. **Test the API** using a tool like Postman or curl.
   - **GET** request to `http://localhost:8080/api/todos` to list todos.
   - **POST** request to `http://localhost:8080/api/todos` with a JSON body to add a todo:
     ```json
     {
       "description": "Learn Spring Boot"
     }
     ```

### Further Enhancements

- **Persistence**: Integrate a database using Spring Data JPA to persist todos.
- **Validation**: Add validation to ensure todos have valid data.
- **Error Handling**: Implement global exception handling for better error responses.

By following these steps, you can convert a basic servlet-based application into a modern Spring Boot REST API, leveraging Spring's powerful features for building scalable and maintainable applications.
