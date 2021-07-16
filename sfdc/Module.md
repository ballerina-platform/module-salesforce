Connects to Salesforce from Ballerina.

## Overview
The Salesforce connector allows users to perform CRUD operations for SObjects, query using SOQL, search using SOSL, and describe SObjects and organizational data through the Salesforce REST API. Apart from these functionalities Ballerina Salesforce Connector includes a listener module to capture events.

## Configuring connector
### Prerequisites
1. Salesforce Organization  
    You can simply setup the Salesforce Developer Edition Organization for testing purposes through the following  link [developer.salesforce.com/signup](https://developer.salesforce.com/signup). 

### Obtaining tokens
1. Visit [Salesforce](https://www.salesforce.com/) and create a Salesforce Account.
2. Create a connected app and obtain the following credentials:
    *   Base URL (Endpoint)
    *   Access Token
    *   Client ID
    *   Client Secret
    *   Refresh Token
    *   Refresh Token URL
3. When you are setting up the connected app, select the following scopes under Selected OAuth Scopes:
    *   Access and manage your data (api)
    *   Perform requests on your behalf at any time (refresh_token, offline_access)
    *   Provide access to your data via the Web (web)
4. Provide the client ID and client secret to obtain the refresh token and access token. For more information on obtaining OAuth2 credentials, go to [Salesforce documentation](https://help.salesforce.com/articleView?id=remoteaccess_authenticate_overview.htm).
5. For event listeners, generate [secret key](https://help.salesforce.com/articleView?id=sf.user_security_token.htm&type=5) and [subscribe](https://developer.salesforce.com/docs/atlas.en-us.224.0.change_data_capture.meta/change_data_capture/cdc_subscribe_channels.htm) channels in the Salesforce dashboard. 

## Quickstart
#### Step 1: Import Ballerina Salesforce module
First, import the `ballerinax/sfdc` module into the Ballerina project.

```ballerina
import ballerinax/sfdc;
```

Instantiate the connector by giving authentication details in the HTTP client config, which has built-in support for OAuth 2.0 to authenticate and authorize requests. The Salesforce connector can be instantiated in the HTTP client config using the access token or using the client ID, client secret, and refresh token.


#### Step 2: Create the Salesforce client
The Ballerina Salesforce connector has allowed users to create the client using the [direct token configuration](https://ballerina.io/learn/by-example/secured-client-with-oauth2-direct-token-type.html) and as well as [bearer token configuration](https://ballerina.io/learn/by-example/secured-client-with-bearer-token-auth.html). 

Users are recommended to use direct-token config when initializing the Salesforce client for continuous access by providing the Salesforce account's domain URL as the `baseURL` and the `client id`, `client secret`, `refresh token` obtained in the step two and `https://login.salesforce.com/services/oauth2/token` as `refreshUrl` in general scenarios. 

```ballerina
// Create Salesforce client configuration by reading from config file.

sfdc:SalesforceConfiguration sfConfig = {
   baseUrl: <"EP_URL">,
   clientConfig: {
     clientId: <"CLIENT_ID">,
     clientSecret: <"CLIENT_SECRET">,
     refreshToken: <"REFRESH_TOKEN">,
     refreshUrl: <"REFRESH_URL"> 
   }
};

sfdc:Client baseClient = new (sfConfig);
```

If the user already owns a valid access token he can initialize the client using bearer-token configuration providing the access token as a bearer token for quick API calls. 

```ballerina
sfdc:SalesforceConfiguration sfConfig = {
   baseUrl: <"EP_URL">,
   clientConfig: {
     token: <"ACCESS_TOKEN">
   }
};

sfdc:Client baseClient = new (sfConfig);
```

This access token will expire in 7200 seconds in general scenarios and the expiration time of the access token can be different from organization to organization. In such cases users have to get the new access token and update the configuration. 

If you want to add your own key store to define the `secureSocketConfig`, change the Salesforce configuration as mentioned below.

```ballerina
// Create Salesforce client configuration by reading from config file.

sfdc:SalesforceConfiguration sfConfig = {
   baseUrl: <"EP_URL">,
   clientConfig: {
     clientId: <"CLIENT_ID">,
     clientSecret: <"CLIENT_SECRET">,
     refreshToken: <"REFRESH_TOKEN">,
     refreshUrl: <"REFRESH_URL"> 
   },
   secureSocketConfig: {
     trustStore: {
       path: <"TRUSTSTORE_PATH"">,
       password: <"TRUSTSTORE_PASSWORD">
      }
    }
};

sfdc:Client baseClient = new (sfConfig);
```

#### Step 3: Implement Operations - SObject Operations
As described earlier Ballerina Salesforce connector facilitates users to perform CRUD operations on SObject through remote method invocations. 

## Snippets
- Create record
The `createRecord` remote function of the baseclient can be used to create SObject records for a given SObject type. Users need to pass SObject name and the SObject record in json format to the `createRecord` function and it will return newly created record Id as a string at the success and will return an error at the failure. 

```ballerina
json accountRecord = {
   Name: "John Keells Holdings",
   BillingCity: "Colombo 3"
 };

string|sfdc:Error recordId = baseClient->createRecord("Account", accountRecord);
```

- Get record
The `getRecord` remote function of the baseclient can be used to get SObject record by SObject Id. Users need to pass 
the path to the SObject including the SObject Id to the `getRecord` function and it will return the record in json at 
the success and will return an error at the failure. 

```ballerina
string testRecordId = "001xa000003DIlo";
string path = "/services/data/v48.0/sobjects/Account/" + testRecordId;
json|Error response = baseClient->getRecord(path);
```

- Update record
The `updateRecord` remote function of the baseclient can be used to update SObject records for a given SObject type. 
Users need to pass SObject name, SObject Id and the SObject record in json format to the updateRecord’ function and it 
will return `true` at the success and will return an error at the failure. 

```ballerina
json account = {
       Name: "WSO2 Inc",
       BillingCity: "Jaffna",
       Phone: "+94110000000"
   };
sfdc:Error? isSuccess = baseClient->updateRecord("Account", testRecordId, account);
```

- Delete record
The Ballerina Salesforce connector facilitates users to delete SObject records by the SObject Id. Users need to pass 
SObject Name and the SObject record id as parameters and the function will return true at successful completion. 

```ballerina
string testRecordId = "001xa000003DIlo";
sfdc:Error? isDeleted = baseClient->deleteRecord("Account", testRecordId);
```

- Convenient CRUD operations for common SObjects
Apart from the common CRUD operations that can be used with any SObject, the Ballerina Salesforce Connector provides 
customized CRUD operations for pre-identified, most commonly used SObjects. They are **Account**, **Lead**, **Contact**, 
**Opportunity** and **Product**. 

Following are the sample codes for Account’s CRUD operations and the other above mentioned SObjects follow the same 
implementation and only the Id should be changed according to the SObject type. 


- Create Account
`createAccount` remote function accepts an account record in json as an argument and returns Id of the account created 
at success. 

```ballerina
json accountRecord = {
   Name: "John Keells Holdings",
   BillingCity: "Colombo 3"
 };

string|sfdc:Error accountId = baseClient->createAccount(accountRecord);
```

- Get Account by Id
User needs to pass the Id of the account and the names of the fields needed parameters for the `getAccountById` remote 
function. Function will return the record in json format at success. 

```ballerina
string accountId = "001xa000003DIlo";

json|sfdc:Error account = baseClient->getAccountById(accountId, Name, BillingCity);
```

- Update Account
`updateAccount` remote function accepts account id and the account record needed to update in json as arguments and 
returns true at success. 

```ballerina
string accountId = "001xa000003DIlo";
json account = {
       Name: "WSO2 Inc",
       BillingCity: "Jaffna",
       Phone: "+94110000000"
   };
sfdc:Error? isSuccess = baseClient->updateRecord(accountId, account);
```

- Delete Account
User needs to pass the Id of the account he needs to delete for the `deleteAccount` remote function. Function will 
return true at success. 

```ballerina
string accountId = "001xa000003DIlo";
sfdc:Error? isDeleted = baseClient->deleteAccount(accountId);
```

- Query Operations
The `getQueryResult` remote function executes a SOQL query that returns all the results in a single response or if it 
exceeds the maximum record limit, it returns part of the results and an identifier that can be used to retrieve the 
remaining results.

```ballerina
string sampleQuery = "SELECT name FROM Account";
SoqlResult|Error res = baseClient->getQueryResult(sampleQuery);
```

The response from `getQueryResult` is either a `SoqlResult` record with total size, execution status, resulting records, 
and URL to get the next record set (if query execution was successful) or Error (if the query execution was unsuccessful).

```ballerina
if (response is sfdc:SoqlResult) {
    io:println("TotalSize:  ", response.totalSize.toString());
    io:println("Done:  ", response.done.toString());
    io:println("Records: ", response.records.toString());
} else {
    io:println("Error: ", response.message());
}
```

If response has exceeded the maximum record limit, response will contain a key named ‘nextRecordsUrl’ and then the user 
can call `getNextQueryResult` remote function to get the next record set. 

```ballerina
sfdc:SoqlResult|sfdc:Error resp = baseClient->getNextQueryResult(<@untainted>nextRecordsUrl);
```

- Search Operations
The `searchSOSLString` remote function allows users to search using a string and returns all the occurrences of the 
string back to the user. SOSL searches are faster and can return more relevant results.

```
string searchString = "FIND {WSO2 Inc}";
sfdc:SoslResult|Error res = baseClient->searchSOSLString(searchString);
```

- Operations to get SObject Metadata
Ballerina Salesforce Connector facilitates users to retrieve SObject related information and metadata through Salesforce 
REST API. Following are the remote functions available for retrieving SObject metadata. 

<table>
  <tr>
   <td><strong>Remote Function</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>describeAvailableObjects
   </td>
   <td>Lists the available objects and their metadata for your organization and available to the logged-in user
   </td>
  </tr>
  <tr>
   <td>getSObjectBasicInfo
   </td>
   <td>Returns metadata of the specified SObject
   </td>
  </tr>
  <tr>
   <td>describeSObject
   </td>
   <td>Returns  metadata at all levels for the specified object including the fields, URLs, and child relationships
   </td>
  </tr>
  <tr>
   <td>sObjectPlatformAction
   </td>
   <td>Query for actions displayed in the UI, given a user, a context, device format, and a record ID
   </td>
  </tr>
</table>

- Operations to get Organizational Data
Apart from the main SObject related functions Ballerina Salesforce Connector facilitates users to get information about 
their organization. Following are the remote functions available for retrieving organizational data. 

<table>
  <tr>
   <td><strong>Remote Function</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>getAvailableApiVersions
   </td>
   <td>Use the <a href="https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_versions.htm">Versions</a> 
   resource to list summary information about each REST API version currently available, including the version, label, 
   and a link to each version's root
   </td>
  </tr>
  <tr>
   <td>getResourcesByApiVersion
   </td>
   <td>Use the <a href="https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_discoveryresource.htm">Resources by Version</a> 
   resource to list the resources available for the specified API version. This provides the name and URI of each additional resource. 
   Users need to provide API Version as a parameter to the function. 
   </td>
  </tr>
  <tr>
   <td>getOrganizationLimits
   </td>
   <td>Use the <a href="https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_limits.htm">Limits resource</a> 
   to list your org limits. 
   </td>
  </tr>
</table>

- Event Listener
The Listener which can be used to capture events defined in a Salesforce instance is configured as below.

```ballerina
sfdc:ListenerConfiguration listenerConfig = {
   username: config:getAsString("SF_USERNAME"),
   password: config:getAsString("SF_PASSWORD")
};
listener sfdc:Listener eventListener = new (listenerConfig);
```

In the above configuration, the password should be the concatenation of the user's Salesforce password and his secret key.

Now, a service has to be defined on the ‘eventListener’ like the following.

```ballerina
@sfdc:ServiceConfig {
    channelName:"/data/ChangeEvents"
}
service quoteUpdate on eventListener {
    resource function onUpdate (sfdc:EventData quoteUpdate) { 
        json quote = op.changedData.get("Status");
        if (quote is json) {
            io:println("Quote Status : ", quote);
        }
    }
}
```

The above service is listening to events in the Salesforce and we can capture any data that comes with it.

### [You can find more samples here](https://github.com/ballerina-platform/module-ballerinax-sfdc/tree/master/sfdc/samples/rest_api_usecases)
