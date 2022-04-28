Available solutions:
- Logback with extra encoder which implements structured logging formats
	- Only available with JVM
		- But most projects run on JVM (Android / Server)
	- Only works with logback directly, so logging facades (SLF4j, Commons Logging) are made irrelevant
- Log4j2 - adding properties using MDC
	- Uses ThreadContext, but should be fine with MDCContext() bridging the gap to coroutines
	- Does not support arbitrarily nested structures, just adding keys/value to a base json object
	- Doesn't really support fully structured logging.. 
- ???

Desired solution:
- Supports different logging implementations through common facades
- Multiplatform capable
- Capable of interfacing with JVM logging standards so Spring users can interoperate?

Can these two goals align? All logging facades are JVM-only?
Does the desired solution mandate creating a Kotlin multiplatform logging facade, which can bind to SLF4J / Commons Logging on the JVM?


