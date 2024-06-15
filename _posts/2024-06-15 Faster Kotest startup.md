---
tags:
  - Kotlin
  - Kotest
date: 2024-06-15
author: Kantis
title: Faster Kotest
---
Add these properties to speed up your Kotest executions.
```kotlin
// in build.gradle.kts or a convention plugin
tasks.withType<Test>().configureEach {  
   useJUnitPlatform()
   systemProperty("kotest.framework.discovery.jar.scan.disabled", "true")  
   systemProperty("kotest.framework.disable.test.nested.jar.scanning", "true")  
   systemProperty("kotest.framework.classpath.scanning.config.disable", "true")  
   systemProperty("kotest.framework.classpath.scanning.autoscan.disable", "true")  
}
```
Doing so dropped my execution time from 2.2s to 600ms when running a single spec using Gradle.

When testing on JVM, Kotest uses ClassGraph to scan for Specs, Kotest configuration, @AutoScan annotations, etc. Disabling these things can have a positive impact on startup time.
## Using Kotest run configurations
The above example only works when tests are executed using Gradle. In some cases, Kotest runs using a Kotest run configuration instead. To apply the same there, you can modify the Kotest run configuration template
![[2024-06-15-config-templates.png]]

![[2024-06-15-vm-options.png]]

Add these VM options:
```
-Dkotest.framework.discovery.jar.scan.disabled=true -Dkotest.framework.disable.test.nested.jar.scanning=true -Dkotest.framework.classpath.scanning.config.disable=true -Dkotest.framework.classpath.scanning.autoscan.disable=true
```

## Using a Kotest config without scanning
Simply add
```kotlin
systemProperty("kotest.framework.config.fqn", "com.foo.MyKotestConfig")
```

Assuming you have a Kotest config that looks like:

```kotlin
package com.foo

class MyKotestConfig : AbstractProjectConfig() {
  // config here...
}
```

## Feedback
Did these changes make any difference to your project, or cause any issues? Reach out to me at [@Kantis@fosstodon.org](https://fosstodon.org/@Kantis), or in #kotest on the [Kotlin language slack](https://kotlinlang.slack.com/) (signup [here](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up)), and let me know.
