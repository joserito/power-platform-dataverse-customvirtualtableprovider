# Custom Virtual Table Provider
This project demostrates how to implement a custom virtual table provider for Dataverse in the Power Plaform.

Official documentation from Microsoft can be found [here](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/virtual-entities/custom-ve-data-providers):

## Scenario
The scenario for this demostration is based on the following assumptions:
- Information from an external system needs to be presented in Dataverse.
- The external system exposes an Http service that follows the REST style.
- The implementation must be configurable in order to configure future resources added to the service.
- Information from the external system is going to be presented as read-only.
## Design
### Dealing with GUID
One of the requirements for adapting an external data source to Dataverse is that the external data source must provide record identifiers in the form of GUID.  A GUID is a 128-bit integer.  When dealing with external systems, they often uses numeric or alphanumeric identifiers, if they can be converted into Guid in C# and they are unique then they integration via this approach can be done.
### Making the custom provider configurable
In order to make the custom virtual table provider configurable, the virtual table definitions stores the name of the resource (REST) serving the data for the virtual table.  Same approach is used for each column in the virtual table.  This information is retrieved during the plugin execution in order to perform the Http call to the necessary resource and once the resource has responded, the column metadata is used to map properties from the JSON object returned as response to the entity attibutes in order to fill the column values the user will see in the user interface.
### Mapping service responses in JSON
External service responses must be standard:
- JSON responses must return an object without other nested objects.
- JSON response object must include the external identifier as one of its properties.
## Components
The following components are required for this demostration:
### Plugin Assembly with 2 Plugin Types (Retrieve and RetrieveMultiple)
Plugin assembly is developed using Visual Studio
### Data Source
### Data Provider
### Virtual Table
