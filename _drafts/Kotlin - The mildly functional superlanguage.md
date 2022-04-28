When picking a language for a new project, it's common to look at a few different metrics. Most people prefer to work with something they're already invested in.

If we are to choose something new, then what would be good criteria to evaluate the platform against? Maybe something like:
1. There should be a pool of talent to tap into for hiring
2. There should be an ecosystem around the language, i.e. frameworks for building applications including:
	1. Configuration management
	2. Observability / Metrics / Tracing
	3. RPC and/or HTTP and/or Messaging
	4. Database connectivity
		1. Transaction management
		2. ORM
		3. Connection pooling
		4. Migration management
	5. Logging
3. Depending on our application architecture, we might be interested in
	1. Binary footprint
	2. Memory footprint
	3. CPU efficiency
	4. Startup times
4. It should be.. fun!

What makes a great language, in itself?
1. Easy to read
2. Easy to write
	1. Comfortable and comprehensible standard library
3. Ability to make invalid states unrepresentable
	1. i.e. we can give our programs compile-time guarantees about data.
	2. Nullability
	3. Typed error-handling
