---
paths:
  - "<source-dir>/**/*.<ext>"
---

# Code Rules

<!-- Replace the language-specific lines. The structure is what transfers: naming,
     dependency injection, boundary discipline, hot-path discipline, failure
     discipline, documentation discipline. -->

- Namespaces/modules mirror the feature folder structure.
- One public top-level type per file; filename matches the type.
- Inject dependencies explicitly through constructors or an initialization call.
  Do not let a class locate its own collaborators.
- Keep event handlers, entry points, and framework callbacks short; delegate to
  testable plain functions and classes.
- Do not use global lookups, string-based dispatch, or ambient discovery in
  production code paths.
- Avoid unobserved asynchronous work. Every async failure must surface somewhere.
- Avoid per-iteration allocations, closures, repeated lookups, and repeated string
  formatting in <name the hot paths>.
- Fail clearly and loudly on invalid input data in development builds. Do not
  silently repair corrupt input at runtime.
- Preserve serialized data when renaming fields or types; use the platform's
  migration/alias mechanism and verify affected data.
- Add documentation comments only where the contract is not evident from naming
  and types.
- A comment that argues *why* a non-obvious choice was made is worth more than ten
  that restate the code. When a reviewer asks "why is this like that", the answer
  belongs in the file, not only in the reply.
