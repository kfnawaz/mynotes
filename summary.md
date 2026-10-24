For edges without a producer ID, should each target table define a fallback process containing all its inbound sources within the same run context/platform?
1. Group by target table
Preserves A→B→C as two processes and combines multi-source transforms into each target.
2. Treat all producerless edges as one 1/O set
One fallback process per environment/connection/node/platform in this invocation.
3. Use another grouping rule
Specify the grouping signal available on CatalogLineageEvent.
4. Enter custom answer