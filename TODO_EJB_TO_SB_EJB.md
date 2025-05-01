### Part 1: Sample EJB Application with iBatis

#### `Todo.java` (Entity)
```java
package com.bank.entity;

public class Todo {
    private Long id;
    private String task;
    private boolean completed;

    // Getters and Setters
}
```

#### `TodoDTO.java` (DTO)
```java
package com.bank.dto;

public class TodoDTO {
    private Long id;
    private String task;
    private boolean completed;

    // Getters and Setters
}
```

#### `TodoMapper.xml` (iBatis Mapper)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//ibatis.apache.org//DTD Mapper 2.0//EN"
  "http://ibatis.apache.org/dtd/ibatis-2.dtd">

<mapper namespace="TodoMapper">
    <select id="getAllTodos" resultClass="com.bank.entity.Todo">
        SELECT id, task, completed FROM Todos
    </select>
    <insert id="insertTodo" parameterClass="com.bank.entity.Todo">
        INSERT INTO Todos (task, completed) VALUES (#task#, #completed#)
    </insert>
</mapper>
```

#### `TodoDAO.java` (DAO)
```java
package com.bank.dao;

import com.bank.entity.Todo;
import java.util.List;

public interface TodoDAO {
    List<Todo> getAllTodos();
    void insertTodo(Todo todo);
}
```

#### `TodoDAOImpl.java` (DAO Implementation)
```java
package com.bank.dao.impl;

import com.bank.dao.TodoDAO;
import com.bank.entity.Todo;
import com.ibatis.sqlmap.client.SqlMapClient;
import java.util.List;

public class TodoDAOImpl implements TodoDAO {

    private SqlMapClient sqlMapClient;

    public List<Todo> getAllTodos() {
        try {
            return sqlMapClient.queryForList("TodoMapper.getAllTodos");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public void insertTodo(Todo todo) {
        try {
            sqlMapClient.insert("TodoMapper.insertTodo", todo);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    
    // Setter for SqlMapClient
}
```

#### `TodoServiceBean.java` (EJB)
```java
package com.bank.ejb;

import javax.ejb.Stateless;
import com.bank.dao.TodoDAO;
import com.bank.entity.Todo;
import java.util.List;

@Stateless
public class TodoServiceBean {

    private TodoDAO todoDAO;

    public List<Todo> getAllTodos() {
        return todoDAO.getAllTodos();
    }

    public void addTodo(Todo todo) {
        todoDAO.insertTodo(todo);
    }
    
    // Setter for TodoDAO
}
```

### Part 2: Convert to Spring Boot

#### `Todo.java` (Entity)
```java
package com.sample.entity;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;

@Entity
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String task;
    private boolean completed;

    // Getters and Setters
}
```

#### `TodoDTO.java` (DTO)
```java
package com.sample.dto;

public class TodoDTO {
    private Long id;
    private String task;
    private boolean completed;

    // Getters and Setters
}
```

#### `TodoRepository.java` (Spring Data Repository)
```java
package com.sample.repository;

import com.sample.entity.Todo;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TodoRepository extends JpaRepository<Todo, Long> {
}
```

#### `TodoService.java` (Service)
```java
package com.sample.service;

import com.sample.entity.Todo;
import com.sample.repository.TodoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class TodoService {
    
    @Autowired
    private TodoRepository todoRepository;

    public List<Todo> getAllTodos() {
        return todoRepository.findAll();
    }

    public void addTodo(Todo todo) {
        todoRepository.save(todo);
    }
}
```

#### `TodoController.java` (Controller)
```java
package com.sample.controller;

import com.sample.entity.Todo;
import com.sample.service.TodoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/todos")
public class TodoController {

    @Autowired
    private TodoService todoService;

    @GetMapping
    public List<Todo> getAllTodos() {
        return todoService.getAllTodos();
    }

    @PostMapping
    public void addTodo(@RequestBody Todo todo) {
        todoService.addTodo(todo);
    }
}
```

### Key Changes in Migration:

1. **Entity Conversion**: EJB entities were retained, but annotations were adjusted for JPA.
2. **DAO to Repository**: iBatis DAO layer has been replaced with Spring Data JPA repositories.
3. **Service Layer**: Converted EJBs into Spring services, using `@Service` and `@Autowired` for dependency injection.
4. **Controller Layer**: Introduced controllers to handle HTTP requests.
5. **Configuration**: Migrated any XML configurations to Java-based configurations if needed or configured via `application.properties`.

This approach illustrates a structured transition from EJB to Spring Boot, emphasizing annotations, Spring Data JPA, and Spring's dependency management.
