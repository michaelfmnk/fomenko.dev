---
title: "Spring Undo v0.0.1 is published!"
date: 2023-02-06
type: "post"
image: "/images/spring-undo.webp"
summary: "I just published Spring Undo v0.0.1. The library is still in development, but you can already try it out, all basic functionality works."
---

# Check out Spring Undo ↩️ v0.0.1!

I just published Spring Undo v0.0.1.

The library is still in development, but you can already try it out, all basic functionality works. Just don't use it for production yet :)

GitHub: [https://github.com/michaelfmnk/spring-undo](https://github.com/michaelfmnk/spring-undo)

Spring Undo is a spring boot starter that provides a way to easily implement undo functionality in your Spring Boot application.

The main features are:
- It drastically simplifies undo implementation. Reduces boilerplate that otherwise, you need to manage.
- It supports application scaling. Persist-module can use shared storage between all instances of your application. So you can call undo from any instance.
- Spring-Undo is a Spring Boot Starter. You don't need to write any additional configuration for setting up Spring-Undo.

#### Usage

Add the following dependencies in your `build.gradle` file:

```gradle
implementation 'dev.fomenko:spring-undo-core:v0.0.1'
implementation 'dev.fomenko:spring-undo-redis:v0.0.1'
```

or `pom.xml` file:

For spring-undo-redis you need to have Redis up & configured in your application.

First thing - you create a simple DTO that represents an event that can be undone.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class EmailChangedEvent {
    private String oldEmail;
    private String newEmail;
}
```

Then register an event listener that will be called when either undo timeout is reached, or undo is called.

```java
@Component
public class EmailChangedUndoEventListener extends UndoEventListener<EmailChangedEvent> {
    @Override
    public void onUndo(EmailChangedEvent action) {
        // handle here undo logic: reset db state
    }

    @Override
    public void onPersist(EmailChangedEvent action) {
        // handle here what to do when event is persisted
        // For example, you can send notification to the new & old email address
    }
}
```

Now autowire `Undo` bean and publish any object that represents an undoable action. `publish` returns an identifier of the action. You can use it to cancel an action.

```java
@RestController
@RequiredArgsConstructor
class UserController {
    private final Undo undo;

    @PutMapping("/email")
    public String changeEmail(ChangeEmailRequest request) {
        // main application logic
        var undoableEvent = new EmailChangedEvent("oldEmail@gmail.com", "newEmail@gmail.com");

        // publish method returns event id that can be used to undo the action
        return undo.publish(undoableEvent);
    }
}
```

And lastly, create a controller that will undo the action by its identifier.

```java
@RestController
public class UndoController {
    private final Undo undo;

    @GetMapping("/undo/{id}")
    public String undo(@PathVariable String id) {
        undo.undo(id);
        return "Undo successful";
    }
}
```

###### You're done!

That's it. Now each time you want to implement an undo, you just create DTO event object & implement listener that will be called by Spring Undo on one of two cases: (1) undo was called, (2) timeout ended & undo was not called.

#### Contacts

Message me if you have any questions, suggestions or ideas for collaboration: [michael@fomenko.dev](mailto:michael@fomenko.dev)

---

©2023 by Mykhailo Fomenko