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
   // [...  maybe other config]
    
   systemProperty("kotest.framework.discovery.jar.scan.disable", "true")    
   systemProperty("kotest.framework.classpath.scanning.config.disable", "true")  
   systemProperty("kotest.framework.classpath.scanning.autoscan.disable", "true")  
}
```
In my project, doing so dropped the test execution time from 2.2s to 600ms when running a single spec using Gradle. Note, this is assuming you're running your tests with Gradle. We'll walk through how to do this with the Kotest runner as well.

### Why?
When testing on JVM, Kotest scans the classpath to find [Kotest configuration](https://kotest.io/docs/framework/project-config.html) and `@AutoScan` annotations. 

> `@AutoScan` is a mechanism which can be used to auto-register Kotest [Extensions](https://kotest.io/docs/framework/extensions/extensions-introduction.html#)
> 
> _Kotest config_ refers to any class that inherits from `AbstractProjectConfig` and configures project-wide Kotest settings. See [these docs](https://kotest.io/docs/framework/project-config.html) for more details
{: .prompt-info}

Classpath scans are costly, so disabling them has a positive impact on startup time. This project is small (~600 classes), and scanning was still the main time sink when running tests. 

> If you're actually using a Kotest configuration, you'll need to help Kotest find it after disabling classpath scanning. See [Using a Kotest config without scanning](#using-a-kotest-config-without-scanning).
{: .prompt-danger}

Discovery jar scan enables finding and running specs from within JARs found on the classpath. This typically does not come into play, but unless you actually care about bundling tests into JARs and running those elsewhere, we might as well disable it.

## Using Kotest run configurations
The above example only works when tests are executed using Gradle. In some cases, Kotest runs using a Kotest run configuration instead. To apply the same there, you can modify the Kotest run configuration template
![An image, showing where to find Run Configuration templates](/assets/2024-06-15-config-templates.png)

![An image showing how to configure the Kotest template's JVM options](/assets/2024-06-15-vm-options.png)

Add these VM options:
```
-Dkotest.framework.discovery.jar.scan.disabled=true -Dkotest.framework.disable.test.nested.jar.scanning=true -Dkotest.framework.classpath.scanning.config.disable=true -Dkotest.framework.classpath.scanning.autoscan.disable=true
```

## Using `kotest.properties`
If Kotest detects a `kotest.properties` file on the classpath, it will load configuration from there, so you can provide the same options using that file. See [docs](https://kotest.io/docs/intellij/intellij-properties.html) for more info.

If your project consists of many modules, it can be tedious to duplicate this configuration in each module, so I personally prefer the Gradle config.
## Using a Kotest config without scanning
As shown earlier, it's really beneficial to disable classpath scanning. Hence, I highly recommend that you help Kotest out by providing the fully-qualified classname of your configuration instead. If you have any config, that is.

To do so, simply add
```kotlin
systemProperty("kotest.framework.config.fqn", "com.foo.MyKotestConfig")
```

Assuming you have a Kotest config class file called `MyKotestConfig` located in the package `com.foo`

### Recognition
Thanks to [OliverO2](https://github.com/Olivero2) and [AJ](https://github.com/ajalt) for reviewing and providing corrections on this article. 

## Feedback
Did these changes make any difference to your project, or cause any issues? Reach out to me at [@Kantis@fosstodon.org](https://fosstodon.org/@Kantis), or in #kotest on the [Kotlin language slack](https://kotlinlang.slack.com/) (signup [here](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up)), and let me know.
