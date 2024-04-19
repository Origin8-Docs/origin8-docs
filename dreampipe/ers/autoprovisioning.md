# Auto Provisioning

Auto provisioning means that you can let your Event-Receiver-Service create the GCP PubSub resources for you, instead of creating them yourself and passing the reference to your ERS.

### Google Cloud Platform Permissions
In order to create your resources, your ERS will need to run with a [Service Account](https://cloud.google.com/iam/docs/service-account-overview) that has the `roles/pubsub.editor` role. See [Pub Sub Roles](https://cloud.google.com/iam/docs/understanding-roles#pub-sub-roles).

Once these resources have been provisioned on initial startup. You can remove the permission until the next time you need to provision new resources.

### Defining your Topic and Subscription Names

You need to explicitly define the pubsub topic and subscription names for the following:

```yaml
application:
  event:
    publisher:
      topics:
        extractionUpdateResultTopicId: projects/myPubsubProject/topics/extractionUpdateResultTopicId
        materializedViewOperationTopicId: projects/myPubsubProject/topics/materializedViewOperationTopicId
    subscriber:
      subscriptions:
        materializedViewOperationSubscriptionId: projects/myPubsubProject/subscriptions/materializedViewOperationTopicId-subscription
```

For your [model configurations](/dreampipe/ers/model_configurations.md), you can explicitly define the topics and subscription names for each entity. Or you can alternatively specify templates.

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
      TranscriptLabel:
        entity:
          kind: transcriptLabelEvents
          materializedViewKind: transcriptLabelsMv
          identifierProperties:
            - transcriptId
          timestampIdentifier: timestamp
          updateTopic: projects/myPubsubProject/topics/pst-Extraction_TranscriptLabel_Update
          updateSubscription: projects/myPubsubProject/subscriptions/psl-Extraction_TranscriptLabel_Update-ERS
          newEntryTopic: projects/myPubsubProject/topics/pst-Extraction_TranscriptLabel_Persisted
          newEntrySubscription: projects/myPubsubProject/subscriptions/psl-Extraction_TranscriptLabel_Persisted_BigQuery-ERS
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

### Cleaning up unused resources

If you change the names of pubsub resources or remove them from the configuration, they will not be removed by the ERS. This is to avoid removing resources defined by another process. These should be removed 