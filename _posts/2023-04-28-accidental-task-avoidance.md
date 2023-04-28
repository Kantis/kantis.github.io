---
title: Accidental task avoidance
author: Kantis
date: 2023-04-28
tags: [Gradle]
---

When I got to work today, my colleague informed me that he couldn't build the `master` branch on his machine.

I checked Github and sure enough, the integration tests on `master` were broken there as well.
It made little sense since we have branch protections in place - it shouldn't be possible to merge a PR where builds are failing.

We were quickly able to conclude which PR had brought in the bad changes based on what failed. Checking the logs revelead that integration tests had simply not run for the PR, even though the `./gradlew check` task was run.

> Task :app:integrationTest FROM-CACHE

![surprised pikachu](/assets/pikachu.webp)
{: .w-25 }

Okay... Why does Gradle think our integration tests are up-to-date?

## Background - Our integration test setup
We package our app into a docker container using [bmuschko's Gradle docker plugin](https://github.com/bmuschko/gradle-docker-plugin).
We then have tests that boot the app in Docker using [Testcontainers](https://github.com/testcontainers).
After starting the app, we use [rest-assured](https://github.com/rest-assured/rest-assured) and JMS to test our application.

> I wanted the integration tests to be completely independent of the implementation of the app itself.
> This gives us the freedom the re-implement large parts of the application without needing to refactor tests constantly.
> In fact, we could even re-implement the application in a different language.
>
> Based on the lifespan of the legacy app we're replacing, it's likely that this application will still be used in production 30 years from now.
{: .prompt-tip}

We separate integration tests from unit tests by defining a `integrationTest` suite using the `jvm-test-suite` plugin.

```kotlin
plugins {
    id("com.bmuschko.docker-spring-boot-application") version "9.3.1"
   `jvm-test-suite`
    // kotlin, spring, etc, omitted for brevity
}

docker {
   springBootApplication {
      baseImage.set("openjdk:17-slim")
      jvmArgs.set(listOf("-XX:+UseContainerSupport"))
      images.set(setOf(applicationImageName))
   }
}

testing {
   suites {
      val test by getting(JvmTestSuite::class) {
         useJUnitJupiter()
      }

      @Suppress("UNUSED_VARIABLE") // Gradle determines name of test suite by the name of the variable
      val integrationTest by registering(JvmTestSuite::class) {
         testType.set(TestSuiteType.INTEGRATION_TEST)
         useJUnitJupiter()

         dependencies {
          // dependencies omitted for brevity (main source set not included)
         }

         targets {
            all {
               testTask.configure {
                  dependsOn(tasks.dockerBuildImage)
                  shouldRunAfter(test)
               }
            }
         }
      }
   }
}
```

## Getting back to the issue
Looking at the code, the only source code changes happened in `./app/src/main`. Given how our setup works, this causes `dockerBuildImage` to run. So, why was a rebuild of the image not enough to invalidate our task?

At this point, I took to the [Gradle community slack](https://gradle-community.slack.com/) for assistance, because I was getting into the deep end of my Gradle knowledge. I figured that if I could somehow tell Gradle that `integrationTest` must run if the main source set changes, then we'd be golden, right?

I gave some background on the issue in Slack and asked how to achieve this without actually depending on the main sources. We don't want access to production code from the integration tests! (Full thread [here](https://gradle-community.slack.com/archives/CAHSN3LDN/p1682667792218029), if you want to read the discussion)

### Depending on files
I figured out how to depend on files after some poking around in the Gradle API.

```kotlin
testTask.configure {
  dependsOn(tasks.dockerBuildImage)
  shouldRunAfter(test)
  inputs.files(fileTree("src/main")) // <--
}
```

Adding this line got us to a point where any change in main sources would cause integration tests to be invalidated and run again.

> There's a caveat to this approach though. If the dockerBuildImage task is changed, we might still skip integration tests!
{: .prompt-danger }

### Using dependsOn
But why would we even need to do this? Surely the docker image is rebuilt when main sources change, and `dependsOn` should make Gradle re-run the integration tests?

As it turns out, no.

`dependsOn` only creates a _lifecycle_ dependency between the tasks. In our case, it only means that integration tests _must run_ after building the docker image. Since the docker image is not an _input_ to the integration tests, Gradle will still avoid running the tests!

![A screenshot from the slack conversation](/assets/depends_on.png)


### Depending on outputs

So instead of using `dependsOn` to create a lifecycle dependency, we want to say that the output of the image task is an input to our integration tests.

Changing our task configuration to the following achieved this.

```kotlin
testTask.configure {
  shouldRunAfter(test)
  inputs.files(tasks.dockerBuildImage.map(DockerBuildImage::imageIdFile)) // <--
}
```

> Vampire also said that since the build image task has a single output, it should be enough to simply use `inputs.files(tasks.dockerBuildImage)` but I ran into some issues with configuration caching when trying that.
{: .prompt-info }

Since we now depend on the output of `dockerBuildImage`, Gradle will figure out the lifecycle dependency implicitly and the `dependsOn` becomes unnecessary.

Gradle can now cache the results of integration tests if our application image is unchanged. Our application image gets rebuilt when sources change. Everything works as intended at this point.

## Kudos
Huge thanks to [Sebastian](https://github.com/sschuberth), [Björn](https://github.com/vampire) and Adam who helped me figure out this ordeal!

Happy Walpurgis!
