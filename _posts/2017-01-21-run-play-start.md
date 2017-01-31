---
layout: post
title: Running code when Play starts
---

Simple snippet that runs during Play 2.5.X initialization. Add your code to the `initialize` method below: 

```
import javax.inject.{Inject, Singleton}

import com.google.inject.AbstractModule
import play.api.inject.ApplicationLifecycle

class Module extends AbstractModule {
  def configure(): Unit = bind(classOf[SystemGlobal]).asEagerSingleton()
}

@Singleton
class SystemGlobal @Inject()(appLifecycle: ApplicationLifecycle) {
  def initialize(): Unit = {

    println("Hello!")

  }

  initialize()
}
```

The `Module` class needs to be in your root package. If you create it anywhere else you have to [register the class](https://www.playframework.com/documentation/2.5.x/ScalaDependencyInjection#programmatic-bindings) in `application.conf`.

You can see slightly more complex example [here](http://stackoverflow.com/a/36455304/848330) with a stop hook that runs on application exit. The Play website also has [good documentation](https://www.playframework.com/documentation/2.5.x/GlobalSettings) about replacing the old `Global` object functionality with dependency injection.
