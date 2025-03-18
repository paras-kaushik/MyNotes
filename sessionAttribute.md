HTTP is indeed stateless, meaning each request is independent and does not retain any information about previous requests. However, sessions provide a way to maintain state across multiple requests from the same client.

### Purpose of `session.setAttribute`

- **State Management**: `session.setAttribute` is used to store data that you want to persist across multiple requests from the same user. This data is stored on the server side and is associated with a unique session ID, which is sent to the client as a cookie.
- **User-Specific Data**: It allows you to store user-specific data, such as login information, user preferences, or temporary data needed for a multi-step process.

### Use in the Given Code

In the provided code, `session.setAttribute` is used to store:

- **`asOfDatesMap`**: A map of dates and their corresponding timestamps.
- **`asOfDates`**: A list of formatted date strings.

These attributes are stored in the session so that they can be accessed in subsequent requests, allowing the application to maintain continuity of data for the user.

### How Sessions Work

1. **Session Creation**: When a user first accesses the application, a new session is created, and a unique session ID is generated.
2. **Session ID**: This ID is sent to the client as a cookie, allowing the server to identify subsequent requests from the same client.
3. **Data Persistence**: Data stored in the session is available across multiple requests as long as the session is active.

### Example Use Case

In a multi-step form submission process, you might store intermediate form data in the session. This way, if the user navigates back and forth between steps, the data is preserved and can be retrieved as needed.

### Considerations

- **Session Timeout**: Sessions have a timeout period after which they expire, and the data is lost.
- **Memory Usage**: Storing large amounts of data in the session can increase memory usage on the server.
- **Security**: Sensitive data should be handled carefully to prevent unauthorized access.


In SpringMVC you can have the HttpSession injected automatically by adding it as a parameter to your method. So, you login could be something similar to:
```java
@GetMapping("/login")
public String login(@ModelAttribute("users") Users user, HttpSession session)
{
    if(userService.authUser(user)) { //Made this method up
        session.setAttribute("username", user.getUsername());
        view.setViewName("homepage"); //Made up view
    }
    else{
        return new ModelAndView("Login");
    }
}
```
