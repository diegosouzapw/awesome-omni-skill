---
name: adding-persistent-event
description: Adds a new type of event that gets persisted to the event log. Use this when adding new kinds of write operations to the system or when adding new events to existing code.
---

# Adding Persistent Event

## Instructions

Copy this checklist and check off items as you complete them. Note that some items are conditional:

```
Task Progress:
- [ ] Create a new sealed interface if needed.
- [ ] Create the event class.
- [ ] Create typealias(es) for the event class.
- [ ] Create a new subject payload class if needed.
- [ ] Make sure the subject is handled in EventLogPayloadTransformer.
- [ ] Add custom FieldsUpdatedAction logic to EventLogPayloadTransformer if needed.
- [ ] Add a database migration to backfill events for existing entities if needed.
- [ ] Add code to publish the new event.
- [ ] Add test(s) for any new logic in EventLogPayloadTransformer.
- [ ] Add test(s) or modify existing tests to verify code that publishes the new event.
- [ ] Generate translations of localized strings with `yarn translate`.
- [ ] Format code with `./gradlew spotlessApply`.
- [ ] Run the full test suite with `./gradlew test`.
```

## Reference

See [EVENTS.md](../../../docs/EVENTS.md) for detailed documentation including best practices.

## Additional guidelines

When implementing `FieldsUpdatedPersistentEvent.listUpdatedFields`, prefer expression syntax over block syntax for the method body, and omit the return type. For example,

    override fun listUpdatedFields() =
        listOfNotNull(createUpdatedField("example", changedFrom.example, changedTo.example))
