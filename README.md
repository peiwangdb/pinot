thirdeye
========

Thirdeye is a system for efficient monitoring of and drill-down into business metrics.

API
---

| Method | Route | Description |
|--------|-------|-------------|
| GET    | `/collections` | Show all collections in server |
| GET    | `/metrics/{collection}` | Aggregate metrics for collection |
| GET    | `/dimensions/{collection}` | Show all explicit values for each dimension |
| GET    | `/configs/{collection}` | Get star-tree configuration for a collection |
| POST   | `/metrics` | Add new record to collection |
| POST   | `/configs` | Add a new star-tree configuration for a collection |
| POST   | `/bootstrap` | Load data from some source into server |

### /metrics

The `/metrics` resource accepts URI query parameters to narrow the scope of the query.

The following are special query parameters reserved for the time dimension:

| Name | Description |
|------|-------------|
| `__BETWEEN__` | A string in the format `{begin},{end}` defining an time range (inclusive) |
| `__IN__` | A CSV list of discrete time buckets |

The rest of query parameters are interpreted as dimension names.

For example, consider a data set `myCollection` that has dimensions `A`, `B`, `C`. One might execute the following query:

```
GET /metrics/myCollection?A=someValue&__BETWEEN__=1000,2000
```

Dimension values can take three special values as well:

| Name | Description |
|------|-------------|
| `*`  | Any value   |
| `?`  | Other value (determined via roll-up algorithm) |
| `!`  | All explicit values (includes `?` but not constitutent values of `?` |

If a dimension is left unspecified in the query string, `*` is assumed.

A more concrete example:

```
>> curl -X GET -s 'http://localhost:8080/metrics/abook?isSuccess=!&browserName=chrome'
[
    {
        "dimensionValues": {
            "browserName": "chrome", 
            "countryCode": "*", 
            "deviceName": "*", 
            "emailDomain": "*", 
            "environment": "*", 
            "errorStatus": "*", 
            "isSuccess": "true", 
            "locale": "*", 
            "source": "*"
        }, 
        "metricValues": {
            "numberOfGuestInvitationsSent": 84129, 
            "numberOfImportedContacts": 219584, 
            "numberOfMemberConnectionsSent": 99804, 
            "numberOfSuggestedGuestInvitations": 433124, 
            "numberOfSuggestedMemberConnections": 217685
        }
    }, 
    {
        "dimensionValues": {
            "browserName": "chrome", 
            "countryCode": "*", 
            "deviceName": "*", 
            "emailDomain": "*", 
            "environment": "*", 
            "errorStatus": "*", 
            "isSuccess": "false", 
            "locale": "*", 
            "source": "*"
        }, 
        "metricValues": {
            "numberOfGuestInvitationsSent": 0, 
            "numberOfImportedContacts": 0, 
            "numberOfMemberConnectionsSent": 0, 
            "numberOfSuggestedGuestInvitations": 0, 
            "numberOfSuggestedMemberConnections": 0
        }
    }
]
```
