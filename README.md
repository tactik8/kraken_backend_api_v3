# kraken_backend_api_v3

## Description
Backend python api for kraken object versioning and storage system.

This backend acts as an api for accessing and storing objects coming from different sources. It allows merging coming from different sources to generate a global record wit hte best information available from each. In the event of conflicting information, criterias are used to decide which to use (see metadata priority). 

### Example for a person record:

Below is an example of a typical data accumulation process. A limited number of fields (keys) and metadata has been kept to increase readability.

The scenario starts with a new email received. The emaila ddress of the sender is used to create a new person object. Then, an API call is made to peopledatalabs.com to retrieve personal information. Another call to gravatar.com is made to retrieve the picture along with some perosnal informaiton. The best information based on the credibility score is kept to producte a final record.

- An email is received in an outlook inbox. The message is created in kraken database and information is extracted, including email of the sender. A person object is created in kraken with only the email address as information.
```
{ 
"@type": "schema:person", 
"schema:email": john.smith@companyxyz.com"
}
```
- An API request is made to peopledatalabs.com to enrich the information. Contact information is added to the person object.
```
{ 
  "kraken:datasource": "peopledatalabs.com",
  "kraken:credibility": 50
  "@type": "schema:person", 
  "schema:email": "john.smith@companyxyz.com",
  "schema:givenname": "John",
  "schema:familyname": "Smith",
  "schema:jobtitle": "Developer",
  "schema:gender": "Male"
}
```

- An API request is made to gravatar.com. A new data point containing picture and limited contact information is added to the person object.
```
{ 
  "kraken:datasource": "gravatar.com",
  "kraken:credibility": 40
  "@type": "schema:person", 
  "schema:email": "john.smith@companyxyz.com",
  "schema:givenname": "John",
  "schema:familyname": "Smith",
  "schema:jobtitle": "Junior developer",
  "schema:image": "https://gravatar.com/john.smith"
}
```

- The person object is compiled, using the best information available from each data point.
```
{ 
  "@type": "schema:person", 
  "schema:email": {
    "value": "john.smith@companyxyz.com",
    "kraken:credibility": 50,'
    "kraken:datasource": "peopledatalabs.com"
    }
  "schema:givenname": {
    "value": "John",
    "kraken:credibility": 50,
    "kraken:datasource": "peopledatalabs.com"
    },
  "schema:familyname": {
    "value":"Smith",
    "kraken:credibility": 50,
    "kraken:datasource": "peopledatalabs.com"
    },
  "schema:jobtitle": {
    "value":"Developer",
    "kraken:credibility": 50,
    "kraken:datasource": "peopledatalabs.com"
    },
  "schema:image": {
    "value":"https://gravatar.com/john.smith",
    "kraken:credibility": 40,
    "kraken:datasource": "gravatar.com"
    }
  }
- The new record object is saved to database, along with each data points.
- When a new data point is generated, relevant fields (keys) are udpated.
```





## Requirements

- Field (key) metadata
  - Each field of an object can have different metadata describing the data itself:
    - value
    - unit of measure (if applicable)
    - data source
    - date created 
    - date modified
    - credibility
- Multiple data points
  - Each object can have multiple data points 
  - Data points can come from same source (a person record from a rCRM that changes over time for example)
  - Data points can come from different sources (a person record with data from a CRM and an ERP)
- Object versioning at the field level
  - Fields for a given object can draw value from different data points depending on best fit.


## Information architecture

### Terminology
- Record: Official record containing all the best information from different data points. A record can have several data points.
- Data point: A series of fields (keys) and values belongiing to a record


### Metadata
For each fields (keys) of a record, the following metadata is also kept:
- value (string, int, float, dict, list, etc): the actual value of the field 
- credibility (int): Number from 0 to 100 representing the credibility / probability of the data being good
- datasource: the system theat generated the data (email, salesforce.com, etc.)
- datasource_date_created: The date the record was created in the original datasource
- datasource_date_modified: The date the record was last modified in the source system
- kraken_date_created: The date the record was originally created in kraken
- kraken_date_modified: The date the record was last modified in kraken

### Metadata priority
When a field (key) is available from different datapoints, the priority shold be given in the following order:
1. credibility
2. datasource_date_created
3. datasource_date_modified
4. kraken_date_created
5. kraken_date_modified

### Object schema
- Objects are stored using schema.org jsonld schema. 
- Fields (keys) should be named using the schema.org schema using the following naming convention: schema:givenname
- Fields (keys) name should be all lowercase


## API endpoints

### /<object>
#### Get
  Parameters:
  - skip: The number of record to skip (for paging)
  - limit: the number of records to retrieve
  - object: the type of record to retrieve (schema:person, schema:product, etc.)
  - term: the terms to search for
  
  Retrieves the list of records and outpus them in a json list.
  
#### Post
  Parameters:
  - payload: record or list of record to store
  - object: the type of record to store
  
  Post a record or list of record to the database
  
#### Delete


## Data processing
### Posting new data points
1. New data point is received through post api call
1. Data point is normalized
    1. keys are put in lowercase
    1. keys are checked for "schema:..." structure. If not, it is added
    1. Nested records are normalized in their own datapoint
1. Sub records are extracted and linked
    1. Sub records nested in data point are extracted and replaced by a link. 
    1. Sub records go through the same process
1. Check for duplicate data point
    1. If datap point already exist, stop process.
1. Record is retrieved from database (reference record)
    1. If record_id is not provided, search base don available uniquely identifiable fields (email, url, etc,)
1. Keep best data
    1. For each fields (key), the new data popint is compared to the reference record
    1. Best fields (keys) is kept 
1. Datapoint is saved to database
1. Save new record (if changed)
1. Return record_id

## Deleting data point
1. Delete data point from database
1. Retrieve all other data point for record
1. Recompile all fields (keys) with data point list
1. Save new record (if changed)

