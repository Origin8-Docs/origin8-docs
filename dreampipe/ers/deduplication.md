# Deduplication

Deduplication in the ERS allows you to emit the same data twice and let the ERS determine if this information has already been stored or not. This allows your updates to be idempotent, and for your event producers to be stateless.

Deduplication verification is performed with transactions. So identical concurrent requests will be de-duplicated correctly. 

## Strategies

### Default  
This will ensure that the new event does not duplicate any historical event that already exists.

### Latest Event Only
This checks that the new event is different to the latest event for that specific entity. This is useful if you don't have an accurate last modified timestamp for your events.

## Which Deduplication strategy should I use?
Generally you should use the default. This allows you to insert events that happened before the latest event currently in the event store. This can be useful when populating the event store from some historical data.

However, in some situations your timestamp is not the last modified timestamp, but the timestamp when the measurement of the entity was taken. This means that new events should only be appended at the end of the event history. With Latest Event Only deduplication, the incoming event is only deduplicated against the entity's latest event. 


