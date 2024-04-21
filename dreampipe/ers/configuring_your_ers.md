# Configuring your Event Receiver Service

Configuration for your ERS is done through configuration parameters. We recommend using your application.yml file to do so.

<details>
<summary><b>Click to see an example application.yml</b></summary>

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

application:
  bigQuery:
    projectId: myGcpProjectId
    dataset: myBigQueryDatasetName
  datastore:
    projectId: myGcpProjectIdForDatastore
#    namespace: myNamespace # Optional
  materializedViewDatastore:
    projectId: myGcpProjectIdForDatastore
#    namespace: myNamespace # Optional
  event:
    publisher:
      topics:
        extractionUpdateResultTopicId: projects/myPubsubProject/topics/extractionUpdateResultTopicId
        materializedViewOperationTopicId: projects/myPubsubProject/topics/materializedViewOperationTopicId # Only required for autoprovisioning
    subscriber:
      subscriptions:
        materializedViewOperationSubscriptionId: projects/myPubsubProject/subscriptions/materializedViewOperationTopicId-subscription
```
</details>


| Parameter                                                                            | Type               | Required?                            | Default Value | Description                                                                                                                                                                                           |
|--------------------------------------------------------------------------------------|--------------------|--------------------------------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `origin8.ers.pubsub.autogenerate`                                                    | Boolean            | No                                   | false         | This will create your GCP PubSub topics and subscriptions if they don't already exist. See [Auto provisioning](/dreampipe/ers/autoprovisioning.md).                                                   |
| `origin8.ers.modelConfigurations`                                                    | ModelConfiguration | Yes                                  | None          | This is where you add your model configurations. See [Model Configurations](/dreampipe/ers/model_configurations.md)                                                                                   |
| `origin8.ers.updateTopicTemplate`                                                    | String             | No                                   | None          | If you use auto provisioning, this template determines the naming pattern of your update topics. See [Auto provisioning](/dreampipe/ers/autoprovisioning.md)                                          |
| `origin8.ers.updateSubscriptionTemplate`                                             | String             | No                                   | None          | If you use auto provisioning, this template determines the naming pattern of your update subscriptions. See [Auto provisioning](/dreampipe/ers/autoprovisioning.md)                                   |
| `origin8.ers.persistedTopicTemplate`                                                 | String             | No                                   | None          | If you use auto provisioning, this template determines the naming pattern of your persisted topics. See [Auto provisioning](/dreampipe/ers/autoprovisioning.md)                                       |
| `origin8.ers.bigQuerySubscriptionTemplate`                                           | String             | No                                   | None          | If you use auto provisioning, this template determines the naming pattern of your Persisted subscriptions used for BigQuery updates. See [Auto provisioning](/dreampipe/ers/autoprovisioning.md)      |
| `origin8.ers.mvSubscriptionTemplate`                                                 | String             | No                                   | None          | If you use auto provisioning, this template determines the naming pattern of your Persisted subscriptions used for Material View updates. See [Auto provisioning](/dreampipe/ers/autoprovisioning.md) |
| `origin8.ers.pubsubGcpProjectId`                                                     | String             | No                                   | None          | If you use auto provisioning, this determines which project will be used to provision your pubsub resources. Will default to current GCP project which is set by the GOOGLE_CLOUD_PROJECT envar       |
| `application.bigQuery.projectId`                                                     | String             | Yes                                  | None          | The GCP project where you wish to store your BigQuery view of your data                                                                                                                               |
| `application.bigQuery.dataset`                                                       | String             | Yes                                  | None          | The dataset name for your BigQuery tables                                                                                                                                                             |
| `application.datastore.projectId`                                                    | String             | Yes                                  | None          | The GCP project for your Event Store                                                                                                                                                                  |
| `application.datastore.namespace`                                                    | String             | No                                   | None          | The namespace used for your Event Store                                                                                                                                                               |
| `application.materializedViewDatastore.projectId`                                    | String             | Yes                                  | None          | The GCP project for your Material Views                                                                                                                                                               |
| `application.materializedViewDatastore.namespace`                                    | String             | No                                   | None          | The namespace used for your Event Store                                                                                                                                                               |
| `application.event.publisher.topics.extractionUpdateResultTopicId`                   | String             | Yes                                  | None          | The name of the PubSub topic where the every success of failure is published (will be auto provisioned if enabled)                                                                                    |
| `application.event.publisher.topics.materializedViewOperationTopicId`                | String             | Only if Auto provisioning is enabled | None          | The topic name used for operations e.g reconciliation or migration (will be auto provisioned if enabled)                                                                                              |
| `application.event.subscriber.subscriptions.materializedViewOperationSubscriptionId` | String             | Yes                                  | None          | The subscription name used for operations e.g reconciliation or migration (will be auto provisioned if enabled)                                                                                       |


