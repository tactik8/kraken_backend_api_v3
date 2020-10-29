# kraken_backend_api_v3

## Description
Backend python api for kraken object versioning and storage system.

This backend acts as an api for accessing and storing objects coming from different sources. 

Example for a person record:
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
    "kraken:credibility": 50
    }
  "schema:givenname": {
    "value": "John",
    "kraken:credibility": 50
    },
  "schema:familyname": {
    "value":"Smith",
    "kraken:credibility": 50
    },
  "schema:jobtitle": {
    "value":"Developer",
    "kraken:credibility": 50
    },
  "schema:image": {
    "value":"https://gravatar.com/john.smith",
    "kraken:credibility": 40
    }
  }
  
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
