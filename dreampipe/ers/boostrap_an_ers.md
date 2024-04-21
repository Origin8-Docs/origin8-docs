# Bootstrap an Event Receiver Service

### Technical Overview

An Event Receiver Service (ERS) is a [Spring Boot](https://spring.io/projects/spring-boot) container and imports DreamPipe's event-receiver-framework Java library as a dependency. This in conjunction with entity configurations will generate everything an ERS needs.

## Creating your ERS

### 1. Create your Spring Boot project
You can initialize a basic Spring Boot project using [Spring Initializr](https://start.spring.io/). We recommend Spring Boot 3.2.4 or above.

### 2. Add the DreamPipe artifactory to your repositories

<details>
<summary><b>Click to see pom.xml example for adding the DreamPipe artifactory repository</b></summary>

```xml
	<build>
		<extensions>
			<extension>
				<groupId>com.google.cloud.artifactregistry</groupId>
				<artifactId>artifactregistry-maven-wagon</artifactId>
				<version>2.2.0</version>
			</extension>
		</extensions>
	</build>

	<repositories>
		<repository>
			<id>artifact-registry</id>
			<url>artifactregistry://us-east1-maven.pkg.dev/prj-cmm-n-build-nqzou69e95/are-usea1-maven-standard-dreampipe-release</url>
			<releases>
				<enabled>true</enabled>
				<updatePolicy>daily</updatePolicy>
			</releases>
			<snapshots>
				<enabled>true</enabled>
				<updatePolicy>always</updatePolicy>
			</snapshots>
		</repository>
	</repositories>
```
</details>

### 3. Add the DreamPipe `event-receiver-framework` dependency

<details>
<summary><b>Click to see pom.xml example for adding the event-receiver-framework dependency</b></summary>

```xml
<dependency>
    <groupId>com.origin8</groupId>
    <artifactId>event-receiver-framework</artifactId>
    <version>7.0.19</version>
</dependency>
```
</details>

Your dependencies should now be complete and look like the below example.
<details> 
<summary><b>Click to view a full pom.xml example</b></summary>

### Example pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.2.4</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.myorganization</groupId>
	<artifactId>my-first-ers</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>my-first-ers</name>
	<description>The microservice responsible for the receiving events and persisting to the event store</description>
	<properties>
		<maven.compiler.source>17</maven.compiler.source>
		<maven.compiler.target>17</maven.compiler.target>
		<java.version>17</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>com.origin8</groupId>
			<artifactId>event-receiver-framework</artifactId>
			<version>7.0.19</version>
		</dependency>
	</dependencies>

	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
		<extensions>
			<extension>
				<groupId>com.google.cloud.artifactregistry</groupId>
				<artifactId>artifactregistry-maven-wagon</artifactId>
				<version>2.2.0</version>
			</extension>
		</extensions>
	</build>

	<repositories>
		<repository>
			<id>artifact-registry</id>
			<url>artifactregistry://us-east1-maven.pkg.dev/prj-cmm-n-build-nqzou69e95/are-usea1-maven-standard-dreampipe-release</url>
			<releases>
				<enabled>true</enabled>
				<updatePolicy>daily</updatePolicy>
			</releases>
			<snapshots>
				<enabled>true</enabled>
				<updatePolicy>always</updatePolicy>
			</snapshots>
		</repository>
	</repositories>
</project>
```
</details>

### 4. Configure your app.yaml

- Rename your application.properties to an application.yml file   
- Add a configuration for your entities.

<details>
<summary><b>Click to see application.yml example</b></summary>

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

application:
  bigQuery:
    projectId: myGcpProjectId
    dataset: myBigQueryDatasetName
  datastore:
    projectId: myGcpProjectIdForDatastore
  materializedViewDatastore:
    projectId: myGcpProjectIdForDatastore
  event:
    publisher:
      topics:
        extractionUpdateResultTopicId: projects/myPubsubProject/topics/extractionUpdateResultTopicId
        materializedViewOperationTopicId: projects/myPubsubProject/topics/materializedViewOperationTopicId # Only required if autoprovisioning is enabled
    subscriber:
      subscriptions:
        materializedViewOperationSubscriptionId: projects/myPubsubProject/subscriptions/materializedViewOperationTopicId-subscription
```

</details>

For in depth information on the app.yaml configuration, please see [Configuring your ERS.](/dreampipe/ers/configuring_your_ers.md)

