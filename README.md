# Custom Virtual Table Provider
This project demostrates how to implement a custom virtual table provider for Dataverse in the Power Plaform.

Official documentation from Microsoft can be found [here](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/virtual-entities/custom-ve-data-providers):

## Scenario
The scenario for this demostration is based on the following assumptions:
- Information from an external system needs to be presented in Dataverse.
- The external system exposes an Http service that follows the REST style.
- The implementation must be configurable in order to allow creating new virtual tables that can be configured to consume a specific resource.
- Information from the external system is going to be presented as read-only in Dataverse.
- External service is OAuth protected.
## Design
### Dealing with GUID
One of the requirements for adapting an external data source to Dataverse is that the external data source must provide record identifiers in the form of GUID.  A GUID is a 128-bit integer.  When dealing with external systems, they often uses numeric or alphanumeric identifiers, if they can be converted into Guid in C# and they are unique then they integration via this approach can be done.  More information about working with GUIDs in C# can be found [here](https://learn.microsoft.com/en-us/dotnet/api/system.guid?view=netframework-4.6.2).
### Making the custom provider configurable
In order to make the custom virtual table provider configurable, the virtual table definitions stores the name of the resource (REST) serving the data for the virtual table.  Same approach is used for each column in the virtual table that needs to be displayed in Dataverse.  This information, known as entity metadata, is retrieved in run-time during the plugin execution in order to perform the Http call to the necessary resource.  Once the resource has responded, the attribute metadata is used to map properties from the JSON object returned as response to the entity attibutes.  This, in order to fill the column values the user will see in the user interface.
### Mapping service responses in JSON
External service responses must be standarized in some way:
- JSON responses must return an object without other nested objects.  Complex object could be used, and nested properties could be mapped, but that would need to be considered in the mapping logic, which will become complex.
- JSON response object must include the external identifier as one of its properties.
### Performance and Caching
Among the things this design is trying to address is avoid unnecessary calls to the external service and if the call must be made, it should be as fast as possible.

When a lookup to a virtual table is created and the lookup column is added to a view or a form, every time the column needs to be displayed is going to trigger the 
Retrieve plugin.  This means, that even if the primary id and primary name attributes are necessary to render the column in the client, the plugin will run.
In order to avoid calling the external service under this scenario, when the plugin run, we are taking a look to the Columns input parameter to determine if is necessary to call the external system.  Primary name attribute is calculated, so it is not necessary to call the external service if the Retrieve plugin runs because the primary identifier and primary name attributes are requested.

Another aspect is caching.  The plugin implementation is relying in entity and attribute metadata in several parts.  In order to avoid calling the Organization service everytime some metadata is needed, a MemoryCache is used.  Same technique can be applied to the HttpClient.
### Lack of query capabilities in the external service
REST service itself does not provide a guidance for enabling query capabilities (e.g. pass a query string expression that could return multiple objects).  So, one way to deal with this is to only allow querying the external system using the external identifier, this will result in 0 or 1 record returned.  Since, the minimal implmementation of a custom virtual table provider requires the implmentation of Retrieve or RetrieveMultiple plugins, the later must be adjusted in order to redirect the request to the logic that performs the query for the Retrieve.
## Components
The following components are required for this demostration:
### Plugin Assembly with 2 Plugin Types (Retrieve and RetrieveMultiple)
### Data Source
### Data Provider
### Virtual Table
