---
title: "HttpStatusDogs is published!"
date: 2022-05-13
type: "post"
image: "/images/httpstatusdogs.webp"
summary: "A few years back I wrote a simple Spring Boot starter that adds a header to your HTTP responses with a link to the appropriate HTTP status dog. The pet project (pun intended) was inspired by Mike Lee's project httpstatusdogs.com. Today I released it on Maven Central!"
---

A few years back I wrote a simple Spring Boot starter that adds a header to your HTTP responses with a link to the appropriate HTTP status dog. The pet project (pun intended) was inspired by [Mike Lee's](https://twitter.com/mike_lee) project [httpstatusdogs.com](https://httpstatusdogs.com).


![HttpStatusDogs](/images/httpstatusdogs.webp)


Today I released it on Maven Central!

#### Usage

You can start using it just by adding it to your dependency list like this for Gradle:

```gradle
implementation 'dev.fomenko:httpstatusdogs:1.0.0'
```

or for Maven:

```xml
<dependency>
    <groupId>dev.fomenko</groupId>
    <artifactId>httpstatusdogs</artifactId>
    <version>1.0.0</version>
</dependency>
```

So now, for any endpoint, you'll have a HttpStatusDog header.

#### Example

Here's an example of a Spring controller:

```java
@RestController
class FakeController {
    @GetMapping("/404")
    public String method404(HttpServletResponse response) {
        response.setStatus(404);
        return "Test";
    }
}
```

And the response it provides:

```
HTTP/1.1 404
StatusDog: https://httpstatusdogs.com/img/404.jpg
Content-Type: text/plain;charset=UTF-8
Content-Length: 5
Date: Fri, 13 May 2022 20:42:44 GMT

Test
```

You can find the code on my GitHub: [https://github.com/michaelfmnk/httpstatusdogs](https://github.com/michaelfmnk/httpstatusdogs)

Maven Central: [https://search.maven.org/artifact/dev.fomenko/httpstatusdogs/1.0.0/jar](https://search.maven.org/artifact/dev.fomenko/httpstatusdogs/1.0.0/jar)

P.S: For me, this lib is an attempt at releasing my own library. I'm planning on releasing my other projects soon.

## Recent Posts

- [How to debug a problem you cannot reproduce](https://www.fomenko.dev/post/how-to-debug-a-problem-you-cannot-reproduce)
- [Check out Spring Undo ↩️ v0.0.1!](https://www.fomenko.dev/post/check-out-spring-undo-v0-0-1)
- [Developing Send-To-Kindle Telegram Bot](https://www.fomenko.dev/post/developing-send-to-kindle-telegram-bot)

---

©2022 by Mykhailo Fomenko

