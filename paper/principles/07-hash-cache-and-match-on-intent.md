# Principle 7: Hash, Cache, and Match on Intent

Route and resolve on semantic meaning, not keywords.

When an orchestrating agent receives a request, it must decide what to do - what data to retrieve, what skills to invoke, what path to take. This decision is a routing problem. And how the routing is implemented has significant consequences for accuracy, performance, and the system's ability to scale.

The naive approach is keyword matching. The request contains the word "invoice," so route it to the invoicing skill. The request mentions "employee," so query the HR data. This works for demos and breaks in production, because natural language is not a controlled vocabulary. Users say "bill" when they mean "invoice." They say "team member" when they mean "employee." They phrase the same intent in ten different ways on ten different occasions. Keyword-based routing is brittle by definition - it matches on surface form rather than meaning.

The alternative is intent-based routing. Understand what the request means - its semantic intent - and route based on that meaning. "Send me the latest bill for Acme Corp" and "Can I see the most recent invoice for Acme?" are different strings. They are the same intent. The system should recognize this and route them identically.

The Three Operations

The principle's name - hash, cache, and match - describes three operations that work together:

Hash. Convert the incoming request into a semantic representation - an embedding, a classified intent, or a structured intent object. This is the act of extracting meaning from surface language. The "hash" is not a cryptographic hash - it is a semantic fingerprint that captures what the request means rather than what it literally says.

Match. Compare the semantic representation against available capabilities - skills in the registry, data domains in the AIL, previously resolved intents in the cache. The matching is semantic, not lexical. "Notify the finance department" matches the notification skill even if the skill is named send_team_alert, because the intent matches the capability.

Cache. When an intent has been resolved - mapped to a specific skill, a specific data source, a specific execution path - cache that resolution. The next time a semantically similar request arrives, the system can reuse the cached resolution rather than re-deriving it from scratch. This improves both performance (faster routing) and consistency (the same intent produces the same routing).

Architectural Implications

Intent-based routing has several consequences for how the system is designed:

The skill registry must be semantically indexed. Skills need more than names and parameter lists. They need descriptions of what they do - in semantic terms that an intent-matching system can compare against incoming requests. A skill described as "Creates a new record in the CRM system for a prospective customer" is matchable against the intent "add this lead to our sales pipeline." A skill described only as crm_create_prospect is not.

The data definition store must support semantic discovery. When an agent needs data to answer a question, it should be able to discover relevant sources based on semantic similarity to the question, not just based on table names or field names. "What's our revenue exposure in APAC?" should route to the appropriate financial and regional data even if no table is named "revenue_exposure" and no field is named "APAC." The memory layer serves as the intent cache. Resolved intents - the mapping from semantic request to execution path - are stored in the memory layer (Principle 3) for fast retrieval. This cache grows over time as more intents are resolved, which means the system's routing becomes faster and more consistent with use. The first time a novel intent is encountered, it requires full reasoning. The hundredth time a similar intent is encountered, it is a cache hit.

Intent classification can be tiered. Not every request needs full semantic analysis. Common, well-established intents can be matched quickly against the cache. Novel or ambiguous intents can be escalated to deeper semantic processing. This tiering reduces the cost of intent-based routing for routine operations while preserving accuracy for novel ones.

The Cache Invalidation Problem

Caching on intent introduces the classical problem: when does a cached resolution become stale?

An intent resolution that mapped "show me Q3 revenue" to a specific query against a specific table was correct when it was cached. If the underlying data source has changed - the table was restructured, the data was migrated, the metric definition was updated - the cached resolution may now be incorrect. The system will confidently route to a stale or broken path.

There is no universal solution to cache invalidation. But two architectural mitigations reduce the risk significantly:

TTL and event-driven expiration. Cached resolutions can expire after a configurable time-to-live, with the TTL reflecting the rate of change of the underlying systems - shorter for volatile domains, longer for stable ones. More precisely, when the AIL's maintenance agents detect changes in source systems - schema updates, data migrations, new sources - they can actively invalidate cached resolutions that depend on the affected systems. The combination of time-based and event-based expiration covers both gradual staleness and sudden structural changes.

Graceful fallback. When a cached resolution fails at execution time - the skill returns an error, the data source is unavailable, the results are inconsistent - the system falls back to full intent resolution rather than simply failing. Combined with Principle 8 (Self-Healing), this means a stale cache entry is automatically corrected on failure rather than requiring manual intervention. The cache is self-healing by default.

Cache invalidation remains a hard problem. Intent-based caching does not solve it - it introduces it. The principle argues that the performance and consistency benefits of intent caching are worth the complexity of managing invalidation, particularly as the system scales and the volume of repeated intents grows.

Tradeoffs and Limitations

Intent-based routing requires an intent understanding layer - which is itself an LLM call, an embedding model, a classifier, or some combination. This creates a bootstrapping concern: you are using intelligence to route to intelligence. The cost and latency of intent classification must be weighed against the benefits of accurate routing.

For small skill registries - a handful of skills with clearly distinct purposes - keyword matching or simple rule-based routing may be sufficient, and the overhead of semantic intent matching is not justified. The principle becomes increasingly valuable as the skill registry and data catalog grow, and as the diversity of incoming requests increases. In early maturity stages, simpler routing may be appropriate. In later stages, intent-based routing becomes a practical necessity.

There is also an accuracy concern. Intent classification is not perfect. Two requests that appear semantically similar may have meaningfully different intents - "show me overdue invoices" and "show me recent invoices" are close in embedding space but functionally distinct. The intent matching layer must be precise enough to distinguish between similar-but-different intents, which requires careful calibration and ongoing evaluation. Cached resolutions for misclassified intents will produce consistently wrong routing - which is worse than inconsistently wrong routing, because it is harder to detect.
