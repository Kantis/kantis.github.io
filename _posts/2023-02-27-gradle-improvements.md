---
title: Improving the default Spring Boot gradle build
author: Kantis
date: 2023-02-26
tags: [Kotlin, Gradle, Spring]
---

## Issues
In the [previous article](./2023-02-26-gradle.md), we broke down the components of the [Spring Initializr](https://start.spring.io) Gradle build. I posted the article to the Kotlin language slack for feedback and Björn Kautler suggested a few improvements we could make.

### Issue 1: Spring dependency management plugin

The plugin filled a niche back in the days when Gradle didn't have support for importing BOMs. Now we can simply import the Spring Boot BOM instead of using the plugin.

```kotlin
dependencies {
  implementation(platform("org.springframework.boot:spring-boot-dependencies:${springBootVersion}"))
}
```

In case we're also applying the Spring Boot plugin, we can get the coordinates for the BOM using a provided property:

```kotlin
dependencies {
  implementation(platform(org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES))
}
```


### Issue 2: Spring configured Kotlin 1.7 by default
According to Sebastien Deleuze, Spring should already be Kotlin 1.8 compatible. Let's configure it instead!

```kotlin
plugins {
  id("org.springframework.boot") version "3.0.3"
  kotlin("jvm") version "1.8.10"
  kotlin("plugin.spring") version "1.8.10"
}
```

This has the added benefit of letting us use improved toolchain support for configuring JVM targets.

### Issue 3: `freeCompilerArgs` overrides previously set args
It's not a problem in the generated starter, but if we had something else in our build adding `freeCompilerArgs` to the Kotlin compiler, these would be lost when we do `freeCompilerArgs = listOf("-Xjsr305=strict")`. We can just add to the list instead by using the `+=` operator.

### Issue 4: The credentials example would expose secrets in build logs
If you need to use authenticated repositories, read up on credential handling [here](https://docs.gradle.org/current/userguide/dependency_management.html#sec:handling_credentials)


## Improved build file

```kotlin
import org.springframework.boot.gradle.plugin.SpringBootPlugin

plugins {
    id("org.springframework.boot") version "3.0.3"
    kotlin("jvm") version "1.8.10"
    kotlin("plugin.spring") version "1.8.10"
}


repositories {
    mavenCentral()
}


dependencies {
    implementation(platform(SpringBootPlugin.BOM_COORDINATES))
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}


kotlin {
    target {
        compilations.configureEach {
            compilerOptions.configure {
                freeCompilerArgs.add("-Xjsr305=strict")
            }
        }
    }

    jvmToolchain {
        languageVersion.set(JavaLanguageVersion.of("17"))
    }
}


tasks.withType<Test>().configureEach {
    useJUnitPlatform()
}
```


## References
1. [Spring BOM](https://stackoverflow.com/questions/31335864/which-one-should-i-use-among-spring-boot-spring-bom-and-spring-io)
2. [Spring - Kotlin 1.8](https://github.com/spring-projects/spring-framework/issues/29754)
