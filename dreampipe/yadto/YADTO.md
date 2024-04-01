# YADTO (Yet Another Data Transfer Object)

When we submit data to the pipeline we need to explicitly specify the data type. We use our own internal protocol written in JSON.

## The YADTO structure

<<<<<<< HEAD:YADTO.md
The structure of any event you send is: Map<String, EntityPropertyValue>

That is to say, it's a map where the keys are strings and the values are an object with the following string fields:
- value
- values
- embeddedEntity
- propertyType

In most cases, you will only use `value` and `propertyType` unless you are using an embedded object or list.
=======
The structure of any event you send is: `Map<String, EntityPropertyValue>`

It's a map where:
- The keys are `strings`
- The values are objects, each containing the following string fields:
    - `value`
    - `values`
    - `embeddedEntity`
    - `propertyType`

For simple properties, you only need to specify the `value` and `propertyType`. If you are working with complex properties 
like embedded objects or lists, additional configuration may be required.
>>>>>>> suggestion:YADTO.md

Here are the [currently available propertyTypes](../src/main/java/com/origin8/eventreceiver/adapters/dto/PropertyType.java).

## Important fields

The following fields should be added to the event payload as important metadata:
```json
{
  "version": {
    "value": "1.0.0",
    "propertyType": "String"
  },
  "eventSource": {
    "value": "origin8cares/my-microservice-name",
    "propertyType": "String"
  }
}
```

You should also have a field that acts an id. It must be unique for the entity it represents. The name should be added to the `identifierProperties` in the [configuration you add in step 1.1](/docs/adding_configurations.md)

```json
{
  "externalIdentifier": {
    "value": "123ABC",
    "propertyType": "String"
  }
}
```

## Example of conversions


## 1. Flat properties 
Consider the following JSON structure:

<details>
<summary><b>JSON code</b></summary>

```json
{
  "name": "Daniel Craggs",
  "age": 21,
  "timeOfBirth": "2002-01-01T00:16:40.000+0000",
  "heightInCm": 209.12
}
```

</details>

To convert this to a YADTO object, we can retain the keys and replace the values with the EntityPropertyValue object:

<details>
<summary><b>YADTO code</b></summary>

```json
{
  "name": {
    "value": "Daniel Craggs",
    "propertyType": "String"
  },
  "age": {
    "value": "21",
    "propertyType": "Integer"
  },
  "timeOfBirth": {
    "value": "2002-01-01T00:16:40.000+0000",
    "propertyType": "DateTime"
  },
  "heightInCm": {
    "value": "209.12",
    "propertyType": "Double"
  },
  "version": {
    "value": "1.0.0",
    "propertyType": "String"
  },
  "eventSource": {
    "value": "origin8cares/my-microservice-name",
    "propertyType": "String"
  }
}
```

</details>

Notice how the version and eventSource metadata have been inserted as additional fields

## 2. List properties

Now let's add a `parts` list to the object. Our JSON object would look like this:

<details>
<summary><b>JSON code</b></summary>

```json
{
    "name": "Daniel Craggs",
    "age": 21,
    "timeOfBirth": "2002-01-01T00:16:40.000+0000",
    "heightInCm": 209.12,
    "parts": [
        "arms",
        "legs",
        "head"
    ]
}
```

</details>

Now lets convert this to YADTO:
<details>
<summary><b>YADTO code</b></summary>

```json
{
  "name": {
    "value": "Daniel Craggs",
    "propertyType": "String"
  },
  "age": {
    "value": "21",
    "propertyType": "Integer"
  },
  "timeOfBirth": {
    "value": "2002-01-01T00:16:40.000+0000",
    "propertyType": "DateTime"
  },
  "heightInCm": {
    "value": "209.12",
    "propertyType": "Double"
  },
  "parts": {
    "propertyType": "List",
    "values": [
      {
        "value": "arms",
        "propertyType": "String"
      },
      {
        "value": "legs",
        "propertyType": "String"
      },
      {
        "value": "head",
        "propertyType": "String"
      }
    ]
  }
}
```

</details>

**Important**!

When specifying a list, the list entries must all be of the same type (in this case they're all strings). Even though YADTO and Datastore do support lists with varying types, BigQuery does not. 

## 3. Embedded Object Properties

Now finally, lets add an embedded object, `pet`. The embedded object will include all the existing attributes we have, but nested inside a map entry. This is supported as Datastore and BigQuery also support it.

Here is the object as JSON:
<details>
<summary><b>JSON code</b></summary>

```json
{
    "name": "Daniel Craggs",
    "age": 21,
    "timeOfBirth": "2002-01-01T00:16:40.000+0000",
    "heightInCm": 209.12,
    "parts": [
        "arms",
        "legs",
        "head"
    ],
    "pet": {
      "name": "Lily",
      "age": "4",
      "timeOfBirth": "2019-01-01T14:49:23.123+0000",
      "heightInCm": 34.56,
      "parts": [
        "paws",
        "legs",
        "head"
      ]
    }
}
```

</details>

And now as YADTO:
<details>
<summary><b>YADTO code</b></summary>

```json
{
  "name": {
    "value": "Daniel Craggs",
    "propertyType": "String"
  },
  "age": {
    "value": "21",
    "propertyType": "Integer"
  },
  "timeOfBirth": {
    "value": "2002-01-01T00:16:40.000+0000",
    "propertyType": "DateTime"
  },
  "heightInCm": {
    "value": "209.12",
    "propertyType": "Double"
  },
  "parts": {
    "propertyType": "List",
    "values": [
      {
        "value": "arms",
        "propertyType": "String"
      },
      {
        "value": "legs",
        "propertyType": "String"
      },
      {
        "value": "head",
        "propertyType": "String"
      }
    ]
  },
  "pet": {
    "propertyType": "EmbeddedObject",
    "embeddedEntity": {
      "name": {
        "value": "Lily",
        "propertyType": "String"
      },
      "age": {
        "value": "4",
        "propertyType": "Integer"
      },
      "timeOfBirth": {
        "value": "2002-01-01T00:16:40.000+0000",
        "propertyType": "DateTime"
      },
      "heightInCm": {
        "value": "34.56",
        "propertyType": "Double"
      },
      "parts": {
        "propertyType": "List",
        "values": [
          {
            "value": "paws",
            "propertyType": "String"
          },
          {
            "value": "legs",
            "propertyType": "String"
          },
          {
            "value": "head",
            "propertyType": "String"
          }
        ]
      }
    }
  }
}
```

</details>

# FAQ
**Why do we need it?**

_When we serialize and de-serialize data into the event-receiver-service, we need to ensure strong typing at the start to ensure we can build structured tables in SQL-like databases such as BigQuery. If you send string that looks like a datetime, we need to ensure it really is a datetime and not a string that just happened to look like one. Otherwise, the downstream database schemas will be created incorrectly._

**Why don't we use JSON, Parquet or Avro instead?**

_Avro and Parquet are designed for different purposes, for example compression. YADTO focuses on explicit definitions for your data. So you can define a data type even with a null value. So we don't rely on type inference._


