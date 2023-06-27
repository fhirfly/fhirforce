# fhirforce
Let's assume you have a custom object in Salesforce called FHIR_Data__c, which represents the data you want to synchronize with the FHIR server. Here's an example of how you can achieve this:

```
trigger FHIRDataTrigger on FHIR_Data__c (after insert, after update, after delete) {
    // Collect the IDs of the records that were inserted, updated, or deleted
    Set<Id> recordIds = new Set<Id>();
    
    if (Trigger.isInsert || Trigger.isUpdate) {
        for (FHIR_Data__c record : Trigger.new) {
            recordIds.add(record.Id);
        }
    }
    
    if (Trigger.isDelete) {
        for (FHIR_Data__c record : Trigger.old) {
            recordIds.add(record.Id);
        }
    }
    
    // Query the FHIR server for the associated data
    // Replace 'FHIR_SERVER_ENDPOINT' with the actual endpoint URL of your FHIR server
    // and 'API_RESOURCE' with the FHIR resource type you want to query (e.g., 'Patient', 'Observation', etc.)
    String fhirServerEndpoint = 'FHIR_SERVER_ENDPOINT';
    String resourceType = 'API_RESOURCE';
    
    // Prepare the request to the FHIR server
    HttpRequest request = new HttpRequest();
    request.setEndpoint(fhirServerEndpoint + '/' + resourceType);
    request.setMethod('GET');
    // Add any required headers or authentication information to the request
    
    // Make the request to the FHIR server
    HttpResponse response = new Http().send(request);
    
    // Process the response from the FHIR server
    if (response.getStatusCode() == 200) {
        // Parse the response body, which contains the queried data
        String responseBody = response.getBody();
        
        // Process the queried data and update the corresponding Salesforce records
        // You'll need to implement the logic to map the FHIR data to Salesforce fields
        
        // Example: Updating the FHIR_Data__c records with the queried data
        List<FHIR_Data__c> recordsToUpdate = new List<FHIR_Data__c>();
        
        for (FHIR_Data__c record : [SELECT Id, Name, Field1__c, Field2__c FROM FHIR_Data__c WHERE Id IN :recordIds]) {
           // Assuming you are querying the 'Patient' resource from the FHIR server

                String responseBody = response.getBody();
                
                // Process the queried data and update the corresponding Salesforce records
                List<FHIR_Data__c> recordsToUpdate = new List<FHIR_Data__c>();
                
                // Iterate over the queried FHIR resources
                FHIR.Bundle bundle = (FHIR.Bundle) FHIR.fromJSON(responseBody);
                for (FHIR.Bundle.Entry entry : bundle.entry) {
                    if (entry.resource.resourceType == 'Patient') {
                        FHIR.Patient patient = (FHIR.Patient) entry.resource;
                        
                        // Create or update a Salesforce record for each Patient resource
                        FHIR_Data__c record = new FHIR_Data__c();
                        
                         FHIR.Patient patient = (FHIR.Patient) entry.resource;
            
            // Create or update an Account and Contact record for each Patient resource
            Account account = new Account();
            Contact contact = new Contact();
            
            // Map FHIR fields to Salesforce fields
            account.Name = patient.name[0].given[0] + ' ' + patient.name[0].family;
            account.Type = 'Customer'; // Set the appropriate Account type
            
            contact.FirstName = patient.name[0].given[0];
            contact.LastName = patient.name[0].family;
            contact.AccountId = account.Id;
            contact.Birthdate = patient.birthDate;
            contact.Gender__c = patient.gender;
            
            accountsToUpdate.add(account);
            contactsToUpdate.add(contact);
        }
                        
                        recordsToUpdate.add(record);
                    }
                }
    
    // Perform the updates on the FHIR_Data__c records
    update recordsToUpdate;
}
            
            recordsToUpdate.add(record);
        }
        
        // Perform the updates on the FHIR_Data__c records
        update recordsToUpdate;
    } else {
        // Handle the case when the request to the FHIR server fails
        // You can log an error, send a notification, or take any other appropriate action
    }
}
```
In this example, we create a trigger on the FHIR_Data__c object that fires after insert, update, or delete events. The trigger collects the IDs of the modified records and then sends a GET request to the specified FHIR server endpoint, querying the desired FHIR resource type. After receiving the response, the code processes the queried data and updates the corresponding Salesforce records.

Please note that this is a basic example, and you will need to customize it according to your specific FHIR server implementation, including authentication, headers, and data mapping logic.

Remember to replace 'FHIR_SERVER_ENDPOINT' and 'API_RESOURCE' with the appropriate values based on your FHIR server configuration and the resource type you want to query.

I hope this example helps you understand how you can monitor Salesforce events and interact with a FHIR server using Apex code.
See more at https://developer.salesforce.com/docs/atlas.en-us.health_cloud_object_reference.meta/health_cloud_object_reference/map_fhir_overview.htm
