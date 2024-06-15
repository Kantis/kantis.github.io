---
tags:
  - Gradle
date: 2024-02-28
author: Kantis
title: Multi-project test report aggregation
---
## Context
We have a multi-project build using multiple jvm test suites (unit tests and integration tests). I want to aggregate the test reports of all these projects, irrespective of test type, into a single report. Googling the issue seemed to yield no results, other than an inconclusive thread.

I asked for advice on the [Gradle community slack](https://gradle-community.slack.com/) and got help from [Björn](https://github.com/vampire) once again (see "Accidental task avoidance" where he helped me last time) 🙂
## Solution
### Define outgoing variants containing all test suites
We create a new test suite type called "all"
```kotlin
// in `buildSrc/src/main/kotlin/uses-message-broker.gradle.kts`

/**
 * Create an outgoing variant for all test results.
 * By default, there's only variants defined per test suite type, so it's impossible
 * to aggregate unit tests with integration tests
 */
val testResultsElementsForAllTests by configurations.consumable("testResultsElementsForAllTests") {
   description = "Directory containing binary results of running tests for all test suite targets."
   isVisible = false
   attributes {
      attribute(CATEGORY_ATTRIBUTE, objects.named(VERIFICATION))
      attribute(TEST_SUITE_TYPE_ATTRIBUTE, objects.named("all"))
      attribute(VERIFICATION_TYPE_ATTRIBUTE, objects.named(TEST_RESULTS))
   }
}
```

And then we add the results from the tests, irrespective of test suite type, to the newly defined configuration.
```kotlin
pluginManager.withPlugin("jvm-test-suite") {
   project
      .testing
      .suites
      .withType<JvmTestSuite>()
      .configureEach { // configure each suite
         targets.configureEach { // not sure what this does
            // add the binary results directory to the "all" results
            testResultsElementsForAllTests.outgoing.artifact(
               testTask.flatMap { it.binaryResultsDirectory },
            ) { 
               type = DIRECTORY_TYPE
            }
         }
      }
}
```

### Aggregating the report
In the aggregating project we start adding the following. 

> 	The aggregating project can be any project in the build. In my case, I chose to put it in the root project.
> {: .prompt-tip}

```kotlin
plugins {
   // other plugins
   `test-report-aggregation`
}
```

Configure the report to use our new "all" variant.
```kotlin
reporting {
   val overallTestReport by reports.registering(AggregateTestReport::class) {
      testType.set("all")
   }
}
```

Add dependencies to all projects you want to include in the report. We can do this for _all_ projects using the following:
```kotlin
dependencies {
   rootProject.allprojects
      // All directories are included in `allprojects`, including those which only hold subprojects
      // These projects have synthetic "build.gradle" files as buildFile, so we can filter on that.
      // Perhaps there's a better way to exclude these projects...
      .filter { it.buildFile.absolutePath.endsWith(".kts") }
      .forEach {
         testReportAggregation(project(it.buildTreePath)) {
            // Don't include transitive dependencies
            setTransitive(false)
         }
      }
}
```