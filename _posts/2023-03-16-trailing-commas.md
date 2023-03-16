---
title: Commas!, Everywhere,
author: Kantis
date: 2023-03-16
tags: [Kotlin, Code style]
---

Trailing commas is a great feature in Kotlin that has been around since version 1.4 (2020).

A trailing comma is a comma left dangling w/o purpose, for instance in this example:

```kotlin
listOf(
    "foo",
    "bar", // <-- trailing comma
)
```


## Why use trailing commas

The primary motivation is to produce cleaner diffs.
If we _didn't_ use the trailing comma in the previous example, and we wanted to add a new element to the list,
we would have to change multiple lines:

```kotlin
listOf(
    "foo",
    "bar", // Comma added
    "baz" //  actual addition
)
```

> 🔖 For a more elaborate example, see the proposal to
> add trailing commas to JavaScript [here](https://github.com/tc39/proposal-trailing-function-commas)


### Why does this matter?
By keeping the versioning history clean, we make it easier for whoever comes digging in our code next time.
They'll immediately see the _actual_ reason for `"bar"` being there, since `blame` will identify the commit
which added `"bar"` instead of our change which just added the comma.

We also make our changes slightly easier to review. Given that there's no cost involved in having the trailing comma,
it's feels like a no-brainer to add it.


## Enforce trailing commas

[Ktlint](https://github.com/pinterest/ktlint) enforces trailing commas by default since `v0.48`. I like to add it as a
Gradle plugin and pre-commit hook, so it runs automatically when Gradle performs verification tasks in CI, and before I
accidentally commit bad code locally.

Note that Ktlint also enforces a much larger set of rules, which provide similar benefits as trailing commas.
