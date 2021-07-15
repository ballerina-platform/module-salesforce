## Overview

Salesforce Bulk API is a specialized asynchronous RESTful API for loading and querying bulk of data at once. This module provides bulk data operations for CSV, JSON, and XML data types.

This module supports Salesforce Bulk API v1 version.
 
## Configuring connector
This is similar to default module. You can refer default module [documentation](https://docs.central.ballerina.io/ballerinax/sfdc/3.0.0).

## Quickstart
#### Step 1: Import Ballerina Salesforce modules
First, import the `ballerinax/sfdc` and `ballerinax/sfdc.bulk` modules into the Ballerina project.

```ballerina
import ballerinax/sfdc.bulk;
```

#### Step 2: Create Salesforce bulk client

```ballerina

sfdc:SalesforceConfiguration sfConfig = {
   baseUrl: <"EP_URL">,
   clientConfig: {
     clientId: <"CLIENT_ID">,
     clientSecret: <"CLIENT_SECRET">,
     refreshToken: <"REFRESH_TOKEN">,
     refreshUrl: <"REFRESH_URL"> 
   }
};

bulk:Client baseClient = new (sfConfig);
```

#### Step 3: Create job and call insert operation

Using the `createJob` remote function of the base client, we can create any type of job and of the data type JSON, XML and CSV. `createJob` remote function has four parameters.

1. Operation - INSERT, UPDATE, DELETE, UPSERT or QUERY
2. SObject type - Account, Contact, Opportunity etc.
3. Content Type - JSON, XML or CSV
4. ExternalIdFieldName (optional) - Field name of the external ID incase of an Upsert operation

Step by step implementation of an `insert` bulk operation has described below. Follow the same process for other operation types too. 

```ballerina
error|bulk:BulkJob insertJob = baseClient->createJob("insert", "Contact", "JSON");
```

Using the created job object, we can add a batch to it, get information about the batch and get all the batches of the job.

```ballerina
   json contacts = [
        {
            description: "Created_from_Ballerina_Sf_Bulk_API",
            FirstName: "Morne",
            LastName: "Morkel",
            Title: "Professor Grade 03",
            Phone: "0442226670",
            Email: "morne89@gmail.com"
        }
    ];
```

Add json content.
```ballerina
    error|bulk:BatchInfo batch = baseClient->addBatch(insertJob, contacts);
```

## Snippets
- Get batch information
```ballerina
    error|bulk:BatchInfo batchInfo = baseClient->getBatchInfo(insertJob, batch.id);
```

- Get all batches
```ballerina
    error|bulk:BatchInfo[] batchInfoList = baseClient->getAllBatches(insertJob);
```

- Get the batch request
```ballerina
    var batchRequest = baseClient->getBatchRequest(insertJob, batchId);
```

- Get the batch result
```ballerina
    error|bulk:Result[] batchResult = baseClient->getBatchResult(insertJob, batchId);
```

- Retrieve all details of an existing job

```ballerina
   error|bulk:JobInfo jobInfo = baseClient->getJobInfo(insertJob);

```
- Close job
The `closeJob` and the `abortJob` remote functions close and abort the bulk job respectively. When a job is closed, no more batches can be added. When a job is aborted, no more records are processed. If changes to data have already been committed, they aren’t rolled back.

```ballerina
  error|bulk:JobInfo closedJob = baseClient->closeJob(insertJob);
```

### [You can find more samples here](https://github.com/ballerina-platform/module-ballerinax-sfdc/tree/master/sfdc/samples/bulk_api_usecases)
