---
title: "How to Debug a Problem You Cannot Reproduce"
date: 2023-03-02
type: "post"
image: "/images/debug.webp"
summary: "Sometimes, I get a bug that I just can't reproduce in the local environment. Still, this bug is very much present in the testing environment (dev or staging). If I can't reproduce it, I can't write a good test to cover this case. And from this point on, bug fixing is just one commit after another with a message 'bug fix #'. Each time you need to wait for the CI/CD pipeline, and after a few tries, your workday is pretty much over. Everybody who has been in this situation knows just how frustrating it might be. I remember how I spent 2-3 hours comparing environments only to realize that there's a bug that appears only on a Docker-container-specific JVM. But what if you could debug code not on your local machine, but remotely? It appears that you actually can! Let's take a look at how to do it with Java. It's not that difficult."
---

Sometimes, I get a bug that I just can't reproduce in the local environment. Still, this bug is very much present in the testing environment (dev or staging).

If I can't reproduce it, I can't write a good test to cover this case. And from this point on, bug fixing is just one commit after another with a message "bug fix #". Each time you need to wait for the CI/CD pipeline, and after a few tries, your workday is pretty much over :(

Everybody who has been in this situation knows just how frustrating it might be. I remember how I spent 2-3 hours comparing environments only to realize that there's a bug that appears only on a Docker-container-specific JVM.

But what if you could debug code not on your local machine, but remotely? It appears that you actually can! Let's take a look at how to do it with Java. It's not that difficult.

In the script where you launch your Java app, you need to pass special parameters:

```bash
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
```

So now your launch command will look something like this:

```bash
java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 app.jar
```

You might want to change `:5005` to the port of your choice. This is the port on which the debug socket will be exposed.

> **WARNING!** This parameter must not be used in production. Remote debug allows remote code execution.

Now you can use your IDE to attach a debugger to your remote running application. You just need to do the last step - add a debug configuration. You can do that in the `Run/Debug Configurations` menu. Choose `Remote JVM Debug` and set the host that your application is running on & the debug socket port.

That's it. Now you can debug code on the staging environment as if it was running locally.

## Conclusion

I highly encourage you to research this topic for a language of your choice, as this ability might save you a lot of time. You can use this to debug not only the app that is running on a different machine but also that is running in Docker locally.

Here are some resources I found for you:

- More on Java remote debugging:
    - [JetBrains tutorial](https://www.jetbrains.com/help/idea/remote-debugging-with-product.html)
    - [DZone article](https://dzone.com/articles/java-remote-debugging)
- [Python remote debugging](https://code.visualstudio.com/docs/python/debugging)
- [C# remote debugging](https://docs.microsoft.com/en-us/visualstudio/debugger/remote-debugging?view=vs-2019)
- [JavaScript remote debugging](https://developers.google.com/web/tools/chrome-devtools/javascript)

---

©2023 by Mykhailo Fomenko