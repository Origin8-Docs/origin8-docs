# <u>Writing Adapters</u>

The adapters are components responsible for extracting data and writing them to the Event Receiver Service. The adapter determines the structure of your entities and describes them explicitly, so they are understood downstream.

For example, you may have pulled the following update from Salesforce for a Lead.
<details>
<summary><b>Expand to view JSON for Lead</b></summary>

```json
{
    "attributes": {
        "type": "Lead",
        "url": "/services/data/v52.0/sobjects/Lead/00Q8G0000321321321"
    },
    "Id": "00Q8G0000321321321",
    "IsDeleted": false,
    "LastName": "lastName",
    "FirstName": "firstName",
    "Name": "firstName lastName",
    "Company": "My Company",
    "Street": "123 Fake Street",
    "City": "Miami",
    "PostalCode": "12345",
    "Country": "United States",
    "Address": {
        "city": "Miami",
        "country": "United States",
        "geocodeAccuracy": null,
        "latitude": null,
        "longitude": null,
        "postalCode": "12345",
        "state": null,
        "street": "123 Fake Street"
    },
  "Phone": "456",
  "MobilePhone": "2345",
  "Email": "myEmail@FakeEmail.com",
  "PhotoUrl": "/services/images/photo/00Q8G0000321321321",
  "LeadSource": "Television Ads",
  "Status": "Protected",
  "OwnerId": "0058G0000123123123",
  "HasOptedOutOfEmail": false,
  "IsConverted": false,
  "ConvertedDate": null,
  "IsUnreadByOwner": false,
  "CreatedDate": "2023-10-27T13:41:08.000+0000"
}
```
</details>

You may wish to pull in a subset of this data, and ... (TBC)


## <u>Writing data directly into DreamPipe</u>

You may have a webservice that generates data and wish to store it in DreamPipe directly. At Origin8, we unify our external and internal sources of data so that our architecture is the same for both. This allows us to replace third party tool functionality with our own seamlessly when we're ready. It also allows us to change vendors easily.

Below is an example of an architecture where two sources are unified. An end user updates a Lead in Salesforce, for example, they update their name and date of birth. The user also adds a label to a transcript of a phone call that occurred for this lead.

Transcript Service has its own web UI which the end user interacts with. It also writes the updates directly to the ERS. There's no database attached directly to the Transcript Service.

In this example the end user adds a label to a transcript of a phone call, and then updates the Lead's name in Salesforce. You can see how the information quickly flows into the same path. And when we read the data, we use the same approach regardless of where the data was originally sourced from.
![unified_write.png](unified_write.png)

The Transcript Service can then read the current state from the Event Query Service as illustrated below: 
![transcript_read.png](transcript_read.png)