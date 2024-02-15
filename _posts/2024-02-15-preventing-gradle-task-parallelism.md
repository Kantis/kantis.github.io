---
tags:
  - Gradle
date: 2024-02-15
author: Kantis
title: Preventing parallelism of certain tasks in Gradle builds
---
## Context
In a multi-module Gradle build, it is sometimes desirable to prevent Gradle from running multiple things in parallel. Gradle typically scans input/output dependencies between tasks to determine which order they can run in and try to run as much as possible in parallel.

In our case, we have some integration tests using system-level resources which are invisible to Gradle. If multiple modules integration tests run in parallel, they would interfere with each other, so we want to indicate to Gradle that these tests can't run together. 

In our particular scenario, we have a message broker running as a docker container on the host machine which is used from integration tests. 

## Solution

Define an empty [Gradle Build Service](https://docs.gradle.org/current/userguide/build_services.html) which limits parallelism. 

The docs for this are fairly complex, and I wanted something that was easy to plug into our builds. We were already using convention plugins to keep our builds DRY, and fortunately there was an easy way of adding a build service this way as well.

This can be done in a convention plugin in `buildSrc` or an included build by adding
```kotlin
// in `buildSrc/src/main/kotlin/uses-message-broker.gradle.kts`
interface MessageBroker : BuildService<BuildServiceParameters.None>

val broker =
   gradle.sharedServices.registerIfAbsent("broker", MessageBroker::class) {
      maxParallelUsages.set(1)
   }

// integrationTest is the name of the task which uses the broker
tasks.named { it == "integrationTest" }.configureEach {
   usesService(broker)
}
```

Now it can be added in the modules using the broker by simply adding 
```kotlin
plugins {
   // other plugins
	id("uses-message-broker")
}
```