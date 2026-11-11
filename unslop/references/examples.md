# Before and after

Examples for the rules that are hard to spot from the one line summary.

## Rule 9. "Not just X, but Y"

Before: The migration is not just a schema change, but a chance to clean up years of drift.

After: The migration changes the schema and cleans up years of drift.

## Rule 13. Em dashes

Before: The parser — which most people never touch — rejects the file.

After: Most people never touch the parser. It rejects the file.

## Rule 14. Colons as connectors

Before: If you're coming from traditional automation: instead of registering event handlers, you describe conditions.

After: Describing when the scheduler should fire works best as plain English.

## Rule 16. Inline header lists

Before:
- **Performance:** Performance improved by cutting queries.
- **Reliability:** Reliability improved by adding retries.

After: Cutting queries improved performance. Adding retries improved reliability.

The lead-in below is fine, because it ends in a period and adds new detail.

After: **Schema in TypeScript.** Tables live in one file.

## Rule 27. Say what it does, not how it feels

Before: The schema stays close at hand, so migrations feel safe.

After: `.toSQL()` returns the exact string sent to the database, so you can check a migration without running it.

Before: types that follow your schema

After: renaming a column fails the build with the name of the column

## Rule 29. Passive voice

Before: Queries are validated by the compiler before they run.

After: The compiler validates queries before they run.

## Rule 32. Mannered prose

Before: The plan holds it, so we ride along and let the dial turn.

After: The plan covers it, so we follow it and vary the retry delay.

## Rule 33. Over-compression

Before: Parser rejects bad date → exit 2, no write

After: The parser rejects a bad date, exits with code 2, and writes nothing.
