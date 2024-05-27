---
tags:
  - Kotlin
  - KotlinConf
  - Conferences
date: 2024-05-27
author: Kantis
title: KotlinConf 2024 Wrapped
---
I went to my first KotlinConf this year. It was a nice experience, and I'd really like to go again next year. It was fun to finally meet face-to-face with some people which I've only interacted with on the Kotlin language slack and GitHub before.

The conference felt big and well organized. Compared to JFokus, it had way less booths and such, but I haven't been to JFokus since before the Covid-19 pandemic which might've changed things I suppose.

Much emphasis was put on Multiplatform and Compose. It feels like JetBrains really want to push us to build end-to-end in Kotlin. As a server-side / backend developer, it felt like I wasn't the intended audience for a lot of things, but it might just be that server-side is very mature already and there's more new things to discuss for Multiplatform/Compose. 
## Amper
The demos of Amper show a lot of promise. Super easy to get started, and the IDE integration makes it a breeze to set up your project. 

Currently you can run Amper as a standalone solution, or use it as a layer on top of Gradle. 

It's not clear yet how Amper aims to solve extensibility concerns, such as:
* custom build steps
* custom source sets

JetBrains said that they have plans on how to configure compiler plugins using standalone Amper.

## Universal KLibs - [KT-52666](https://youtrack.jetbrains.com/issue/KT-52666/Kotlin-Multiplatform-libraries-without-platform-specific-code-a.k.a.-Pure-Kotlin-libraries-Universal-libraries)
I think this is a big deal. Currently it's a big hassle to set up builds for all Kotlin targets, if you're mainly focused on writing re-usable Kotlin code. With this improvement, the Kotlin ecosystem is more likely to see a surge in multiplatform-capable open-source projects.

## Non-local returns/breaks
This has been available under an experimental flag IIRC, but it's fantastic to see it being promoted into stable. 

## `when` with guards
I think it's nice to see the when-statements gain more capabilities. I just wish the syntax didn't look so contrived. 

The given example was something roughly like:

```kotlin
when (something) {
  is Blockable if something.isBlocked -> println("foo")
  is Blockable -> println("bar)
  is NotBlockadle -> println("baz")
}
```

There was a question regarding why they opted for `if` instead of `&&`, but I can't recall the  answer. 

I wonder if something like the following could be possible instead:

```kotlin
when (something) {
  is Blockable(isBlocked = true) -> println("foo")
  is Blockable -> println("bar)
  is NotBlockadle -> println("baz")
}
```


## `dataargs` for long optional parameter lists
`dataargs` was a suggested solution that JetBrains is looking into to prevent library authors from having to duplicate functions when adding new optional parameters.

Code compiled to use a certain version of a library will break when the method signature changes, even if that change is just adding a new parameter with a default value. I didn't quite understand how the `dataargs` class helped. 🤔

## Union types for errors
Michail Zaracenskij mentioned the union types for errors in his talk, and Ross Tate dove a lot deeper into the reason for limiting union types to errors. 

In short, the main reason was that type-checking runs in exponential time if we allow arbitrary union types. By limiting unions to just errors, it can still be performed in polynomial time.

Proposed syntax is mark errors types with the `error` keyword, and to use similar operators that we're already used to when dealing with nullability, but with `!` instead of `?`, such as:
```kotlin
val x = sequence.lastOrError { it.value == 5 }
return x!.value !: error("Failed to find element with value 5")
```

It's not clear how we'll deal with nullability and errorability together, will we just mash it all into `?!.`? 

Overall I'm not a fan of the operators `!.` and `!:`.  Seeing bangs in code is currently typically a code smell, which this proposal would normalize. I also think that `!!.` and `!.` are way too similar, while achieving drastically different things (`unpack or throw`, vs `safe unpack`)

## Summary
Some very appreciated changes making it in with Kotlin 2.0, but the future is a bit uncertain. I hope that whatever design is settled is thought through as it'll very likely stay with us for many years. Overall I'm not a big fan of `dataargs` and `error`-types, both which seem to add complexity to the language in order to solve too niche problems.

I hope that whatever design is adopted for union types can be re-used in other ways. 
