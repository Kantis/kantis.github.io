# Ktlint problems

## Block comments should be preceded by a blank line

Current:
```kotlin
data class PipedriveWebhookConfig(
   val loggingEnabled: Boolean,
   /**
    * Map of Pipedrive hash -> [Deal] property name
    */
   val dealFields: Map<String, String>,
   /**
    * Map of Pipedrive hash -> [Organization] property name
    */
   val organizationFields: Map<String, String>,
)
```

Desired:
```kotlin
data class PipedriveWebhookConfig(
   val loggingEnabled: Boolean,

   /**
    * Map of Pipedrive hash -> [Deal] property name
    */
   val dealFields: Map<String, String>,

   /**
    * Map of Pipedrive hash -> [Organization] property name
    */
   val organizationFields: Map<String, String>,
)
```


## Right-hand side of assignment should start on same line, when possible

### Non-conformant code
```kotlin
val x =
  5

val y = 
  SomeClass(
    "Foo",
    "Bar"
  )
```


### Conformant code
```kotlin
val x = 5

val y = SomeClass(
  "Foo",
  "Bar",
)
```

### Motivation:

Letting the assignment start on the same line means we don't introduce an extra level of indentation. The assignment starts and ends on the same level of indentation, so it's easier to visually scan for the end of the assignment.

## Class-signature supertype should be on same line if possible

```
class Foo : Bar // ✅

class Foo :
  Bar // 🚫
```

## Trailing commas should only be enforced for already comma-delimited callsites

Primarily an issue for Kotest, IMO

```kotlin
class FooSpec : StringSpec({
  "Bar" {
  
  }
})
```

is forced into
```kotlin
class FooSpec : 
	StringSpec( // by class-signature rule
	   { // by trailing-comma rule
	      "Bar" {
      
	      }
	   },
	)
```
a
