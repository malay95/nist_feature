# NIST Feature

## Problem Statement

As a system, we need to process a list of CVEs to extract NIST feature data. For each CVE build a
list CWEs and a list of CPEs related to the CVE. Once you have processed the entire list of CVEs,
store the extracted feature data in the FeatureData collection. So that we can use the FeatureData
collection to build the NIST feature. 


## Overall Acceptance Criteria

    Given: A list of CVE strings.   
    When: The NIST feature is ran.  
    Then: All documents in the cve list are retrieved from MongoDB  
    Then: The documents are processed and the feature data is extracted from the documents.  
    Then: The feature data is updated in the FeatureData collection.

## Persistence class Acceptance Criteria:
The Persistence class that will be the only class to interface with MongoDB. You will use this class
to query the CVEInfo collection and update the FeatureData collection. 

When you are updating the FeatureData collection:
- if the document in FeatureData collection does not have a field that is in the new feature data
dictionary then simply insert it. 
- if the document in FeatureData collection does have the field, then update the metadata.
When updating the respective list ensure that we are only storing unique values.
- if a cve is missing either cwe_info, cpe_info, or both then you will insert default values.
(See domain knoweldge for document schemas)
- if a cve has None as the value, then insert default values.


        Given: a list of cves to process
        When: get_cveInfo_data function is called
        Then: a list of all the documents that have a cve in the list of cves is returned

        Given: a dictionary that has the cve as key and the feature data as a value
        Given: the cve does not have a document in the FeatureData collection
        When: update_FeatureData_collection funciton is called
        Then: the feature data is inserted into the FeatureData collection

        Given: a dictionary that has the cve as key and the feature data as a value
        Given: the cve already has a document in the FeatureData collection
        When: update_FeatureData_collection function is called
        Then: the new feature data is added to the existing document in the FeatureData collection
        

## NISTInfo class Acceptance Criteria:
The NISTInfo class will be used to process documents and create a dictionary with new CWE / CPE
info. The run function will act as a wrapper function for the entire class. The run class will get
the data, call the process_documents function, and then update the FeatureData collection. 

The main function in the NISTInfo class is the process_documents function. This will iterate through
the documents, extract the pertinent data, and add it to an overall update dictionary.  

When processing the individual documents: 
- if the document is missing the cwe or cpe field then simply don't add the missing field to the
update_dictionary.
- if the document is missing both fields then add the cve to the dictionary and set the value to
None. In the Persistence.update_FeatureData_collection function will need to handle this.
  
        Given: a list of cves  
        When: the run function is called  
        Then: get the documents to process   
        Then: process the documents to create   
        Then: update the documents in FeatureData collection  


        Given: a list of cves  
        When: get_data is called  
        Then: use the persistence class to get the list of documents to process  

        Given: a list of documents to analyze
        When: process_documents funciton is called
        Then: process each document to extract the cwe_info and cpe_info from the 
        Then: a dictionary is returned that has the cve as key and a dictionary with the feature
        data as the value 

        Given: a dictionary with values to update
        When: update_feature_data is called 
        Then: use the persistence class to update the FeatureData collection

## Notes:
1. You will need to create two classes. A  and a NISTInfo class that will process the documents and
create a dictionary with the feature data. The NISTInfo class will use the classes in the Persistence
class to query and update MongoDB.
2. We have provided a few unit tests to check the overall functionality of the two classes. You are
expected to write additional unit tests for any functions that you create and to test additional
scenarios. 
3. Please review the below information to provide additional domain knowledge for this story. 


# Domain Knowledge

## Environment Notes:
1. Before starting to code, install the dependencies in the requirements.txt file. 

2. We are using MongoMock to simulate the mongoDB. MongoMock is an in-memory database that has all
the functionality of mongoDB. We have pre-loaded mocked data that you can use for testing. 

3. If you need to mock any objects or functions use the mock library. 

4. We are using pytest as our testing framework. 

## CVEInfo Collection
We have the NIST data stored in a collection called CVEInfo. There is a many to one mapping for
NIST documents to cve. Each document in CVEInfo should have the following fields:  



| Field | Type | Default | Description |
| --- | --- | --- | --- |
| cve | String | N/A | the CVE number used to identify the vulnerability. All cves will be composed of lower case letters and numbers.| 
| website_name | String | N/A | Will either be NIST or NISTJsonFeed |
| cwe | List | Not present in document | a list of CWE strings assigned to a particular cve | 
| cpe | List | Not present in document | a list of CPE strings assigned to a particular cve | 
|description | String | Empty String | a string containing the description for the cve |
| recorded_date | bson.Date | N/A | the date the information was retrieved from the website| 


An example document from the CVEInfo collection is:
```json
{
    "cve": "cve-123",
    "website_name": "NIST", 
    "cwe": ["CWE-79", "CWE-80", "CWE-129"], 
    "cpe": ["cpe:/a:apache:struts:2.3.8", "cpe:/a:apache:struts:2.1.5"],
    "description": "This is a description for cve-123",
    "recorded_date": "2014-03-11T00:00:00Z"
}
```
## FeatureData Collection
In the FeatureData collection we need to store the following fields for each cve:

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| CVE | String | N/A | the CVE number used to identify the vulnerability. All cves will be composed of lower case letters and numbers.| 
| cpe_info | Dictionary | Dictionary with default fields | a dictionary containing metadata for CPEs |
| cpe_info.source | String | None | the website that the information was retrieved. Will either be NIST or NIST_JsonFeed | 
| cpe_info.last_updated_date | bson.Date | None | the date when the cpe_info was last updated | 
|cpe_info.cpe_list | List | Empty List | a list of unique CPEs associated with the CVE. |
| cwe_info | Dictionary | Dictionary with default fields | a dictionary containing metadata for CWEs | 
| cwe_info.source | String | None | the website that the information was retrieved. Will either be NIST or NIST_JsonFeed | 
| cwe_info.last_updated_date | bson.Date | None | the date whenthe cpe_info was last updated | 
|cwe_info.cwe_list | List | Empty List | a list of unique CWEs associated with the CVE. |

FeatureData collection example:
```json
{
    "cve": "cve-123",
    "cpe_info": {
        "source": "NIST",
        "last_updated_date": "2014-03-11T00:00:00Z",
        "cpe_list": [
            "cpe:/a:apache:struts:2.3.8",
            "cpe:/a:apache:struts:2.1.5",
            "cpe:/a:apache:struts:2.0.3",
            "cpe:/a:apache:struts:2.2.1",
            "cpe:/a:apache:struts:2.0.7",
            "cpe:/a:apache:struts:2.3.15",
            "cpe:/a:apache:struts:2.0.11",
            "cpe:/a:apache:struts:2.3.4.1"
        ]
    }, 
    "cwe_info": {
        "source": "NIST_JsonFeed", 
        "last_updated_date": "2014-03-12T00:00:00Z",
        "cwe_list": [
            "CWE-264", 
            "CWE-119"
        ]
    }  
}
```
