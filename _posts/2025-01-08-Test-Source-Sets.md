---
tags:
  - Gradle
  - Kotlin
  - TestingStrategy
date: 2025-01-08
author: Kantis
title: Adding additional test source sets
---
It's quite common to have different types of tests. Many have been taught that one should strive for a testing pyramid. Something like:
![[Pasted image 20250108230927.png]]
> The reason for the pyramid shape is to convey the relative weight that should be placed on each category.  See [Martin Fowler's post](https://martinfowler.com/bliki/TestPyramid.html) for more reading.
> 
> I don't fully agree (definitely a topic for another post), but I certainly see some value in dividing tests into some categories. 
{: .prompt-tip}

Given that we have different types of tests, which might differ by orders of magnitude in execution time and have different build requirements, it makes sense to divide them.o

We can of course keep all tests in a single source set and categorise them using `@Tags` or some similar approach, but I think there are a few advantages of dividing them. Namely:

 1. You can have separate static testing setups (setup/teardown hooks, other lifecycle extensions) for the different test categories
 2. You can add build step requirements for running the `integration tests`, such as building your app into a Docker image first.
 3. It's easier to know what's what, and not get things mixed up and messy.

## Creating a custom test source set

Gradle comes with a plugin out-of-the-box called [jvm-test-suite](https://docs.gradle.org/current/userguide/jvm_test_suite_plugin.html) which lets us define source sets in a semantic way that's also compatible with _variants_ and supports aggregation of test reports. It was released in [Gradle 7.3.1, December 2021](https://docs.gradle.org/7.3.1/release-notes.html#new-features-and-usability-improvements) so don't worry about the `Incubating API` too much.

Sigh. I started writing an example here, but for once the [Gradle docs](https://docs.gradle.org/current/userguide/jvm_test_suite_plugin.html#sec:declare_an_additional_test_suite) explained it quite well so just go read there. 