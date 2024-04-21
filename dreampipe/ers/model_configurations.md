# Model Configurations

Your Model Configurations contain your Entity Configuration which describes the Pub Sub topics and subscriptions that they are attached to. The Entity Configurations also describe crucial properties of your entities. These are described in detail further below.

## Defining your PubSub resources
You can explicitly define the topics and subscription names for each entity. Or you can alternatively specify templates.

Below is an example where we define everything explicitly:
```yaml
origin8:
  ers:
    pubsub:
      autogenerate: true
    modelConfigurations:
##### Entity Configs
      SalesforceLead:
        entity:
          kind: salesforceLeadEvents
          materializedViewKind: salesforceLeads
          identifierProperties:
            - id
          timestampIdentifier: timestamp
          updateTopic: projects/myPubsubProject/topics/pst-Extraction_SalesforceLead_Update
          updateSubscription: projects/myPubsubProject/subscriptions/psl-Extraction_SalesforceLead_Update-ERS
          newEntryTopic: projects/myPubsubProject/topics/pst-Extraction_SalesforceLead_Persisted
          newEntrySubscription: projects/myPubsubProject/subscriptions/psl-Extraction_SalesforceLead_Persisted_BigQuery-ERS
          materializedViewUpdateSubscription: projects/myPubsubProject/subscriptions/psl-Extraction_SalesforceLead_Persisted_MaterializedView-ERS
      TranscriptLabel:
        entity:
          kind: transcriptLabelEvents
          materializedViewKind: transcriptLabelsMv
          identifierProperties:
            - transcriptId
          timestampIdentifier: timestamp
          updateTopic: projects/myPubsubProject/topics/pst-Extraction_TranscriptLabel_Update # Required for auto provisioning only
          updateSubscription: projects/myPubsubProject/subscriptions/psl-Extraction_TranscriptLabel_Update-ERS
          newEntryTopic: projects/myPubsubProject/topics/pst-Extraction_TranscriptLabel_Persisted
          newEntrySubscription: projects/myPubsubProject/subscriptions/psl-Extraction_TranscriptLabel_Persisted_BigQuery-ERS
          materializedViewUpdateSubscription: projects/myPubsubProject/subscriptions/psl-Extraction_TranscriptLabel_Persisted_MaterializedView-ERS
##################################################################################################################
```

And here is an effectively identical configuration using templates instead:
```yaml
origin8:
  ers:
    pubsub:
      autogenerate: true
    modelConfigurations:
##### Entity Configs
      SalesforceLead:
        entity:
          kind: salesforceLeadEvents
          materializedViewKind: salesforceLeads
          identifierProperties:
            - id
          timestampIdentifier: timestamp
      TranscriptLabel:
        entity:
          kind: transcriptLabelEvents
          materializedViewKind: transcriptLabelsMv
          identifierProperties:
            - transcriptId
          timestampIdentifier: timestamp
##################################################################################################################
    updateTopicTemplate: "pst-Extraction_${entity}_Update"
    updateSubscriptionTemplate: "psl-Extraction_${entity}_Update-ERS"
    persistedTopicTemplate: "pst-Extraction_${entity}_Persisted"
    bigQuerySubscriptionTemplate: "psl-Extraction_${entity}_Persisted_BigQuery-ERS"
    mvSubscriptionTemplate: "psl-Extraction_${entity}_Persisted_MaterializedView-ERS"
    pubsubGcpProjectId: myPubsubProject # Optional, will use default GCP project defined by the envar GOOGLE_CLOUD_PROJECT
```

### Auto provisioning
You can create your pubsub topics yourself and then pass the references to the configuration, or you can let the provision these for you dynamically instead. Please see [Auto Provisioning](/dreampipe/ers/autoprovisioning.md)

## Defining your entities
This is a crucial part of your data design. The Event-Receiver-Service needs to know important attributes of your data so that it records new information as you expect.

Below is a configuration that uses all currently available settings, and a table that describes each parameter.
```yaml
origin8:
##### Entity Configs
      SalesforceLead:
        entity:
          kind: salesforceLeadEvents
          materializedViewKind: salesforceLeads
          calculatedFieldCalculator: leadCalculator
          materializedViewProperties:
            - leadPhone
            - email
            - id
          ignoreableProperties:
            - country
            - city
          deduplicationMode: DEFAULT
          additionalIndexProperties:
            - leadPhone
            - email    
          messageAttributeProperties:
            - leadPhone
            - email
          identifierProperties:
            - id
          timestampIdentifier: timestamp
##################################################################################################################
```

| Entity configuration field   | Type         | Required? | Default Value | Description                                                                                                                                                                                                                             |
|------------------------------|--------------|-----------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `kind`                       | String       | Yes       | false         | This determines the kind name that will be used when for the Event Store of this entity in Datastore. This is similar to the table name in a traditional database                                                                       |
| `materializedViewKind`       | String       | No        | N/A           | This determines the kind name that will be used when for the Material View of this entity in Datastore. This is similar to the table name in a traditional database                                                                     |
| `calculatedFieldCalculator`  | String       | No        | N/A           | This specifies that you wish to do add some custom fields to your entity in the material view. See [Material Views](/dreampipe/ers/material_views.md)                                                                                   |
| `materializedViewProperties` | List<String> | No        | N/A           | Specifies the fields you wish to be added to your material views. See [Material Views](/dreampipe/ers/material_views.md)                                                                                                                |
| `ignoreableProperties`       | List<String> | No        | N/A           | Specifies which fields are not to be considered when deduplicating. See [Deduplication](/dreampipe/ers/deduplication.md)                                                                                                                |
| `deduplicationMode`          | String       | No        | DEFAULT       | Defines your deduplication strategy. See [Deduplication](/dreampipe/ers/deduplication.md)                                                                                                                                               |
| `additionalIndexProperties`  | List<String> | No        | N/A           | This lets you specify extra fields you wish to be indexed in the Datastore Event Store. In most cases this is not required.                                                                                                             |
| `messageAttributeProperties` | List<String> | No        | N/A           | Adds the listed fields into the PubSub message attribute headers when publishing to the Persisted/New Entry topic. Useful when you wish to use [subscription filters](https://cloud.google.com/pubsub/docs/subscription-message-filter) |
| `identifierProperties`       | String       | Yes       | N/A           | Defines which field(s) should be used to determine to match an incoming event with any potential existing event. Described in more detail below                                                                                         |
| `timestampIdentifier`        | String       | Yes       | N/A           | Defines which field represents the timestamp of your event. Described in more detail below                                                                                                                                              |

### Identifier Properties
When the ERS receives an event, it needs to know which field is its identifier. Otherwise, it doesn't know how to match it with other events in the history. This is typically some field you know to be unique to the entity.

The ERS will generate a uuid for your entity. This is to ensure that we have a completely unique identifier across the entire datastore. You can also rely on this to generate an ID for you if you know which combination of properties should be unique.

In those cases, you can simply add all the properties that defines a unique entity to the identifierProperties field.

### Timestamp Identifier
One of your properties on your entity must be a timestamp. And this timestamp represents when your entity was last modified. This allows you to populate insert historical data into your event store if you have it.

Sometimes it can be difficult to determine an accurate last modified timestamp. This can have adverse effects on deduplication. In these cases you should refer to the full section on [Deduplication](/dreampipe/ers/deduplication.md).