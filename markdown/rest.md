# Table of Content
* [Features](#features)
* [APIs overview](#api-overview)
* [Authentication](#authentication)
* [Access Control](#acess-control)
* [Migrations](#migrations) 

# Features
* **Automatic REST APIs for any SQL database** 
    * Generates REST APIs for **ANY** MySql, Postgres, MSSQL, Sqlite database :fire:
    * Serves APIs irrespective of naming conventions of primary keys, foreign keys, tables etc :fire:
    * Support for composite primary keys :fire:
    * REST APIs : 
        * CRUD, List, FindOne, Count, Exists, Distinct (Usual suspects)
            * Pagination 
            * Sorting
            * Column filtering - Fields :fire:  
            * Row filtering - Where :fire:
        * Bulk insert, Bulk delete, Bulk read :fire:   
        * Relations - automatically detected
        * Aggregate functions
    * More
        * Upload single file
        * Upload multiple files
        * Download file
* **Authentication**
* **Access Control**


# API Overview

|  **Method**  |  **Path**  | **Query Params** | **Description**  |
|---|---|---|---|
|  **GET** | [/api/v1/tableName](#list)  | [where, limit, offset, sort, fields](#query-params) | List rows of the table |
|  **POST** | [/api/v1/tableName](#create)  | | Insert row into table |
|  **PUT** | [/api/v1/tableName/:id](#update)  | | Update existing row in table |
|  **GET** | [/api/v1/tableName/:id](#get-by-primary-key)  | | Get row by primary key |
|  **GET** | [/api/v1/tableName/:id/exists](#exists)  | | Check row with provided primary key exists or not |
|  **DELETE** | [/api/v1/tableName/:id](#delete)  | | Delete existing row in table |
|  **GET** | [/api/v1/tableName/findOne](#find-one)  | [where, limit, offset, sort, fields](#query-params) | Find first row which matches the conditions in table |
|  **GET** | [/api/v1/tableName/groupby/:columnName](#group-by) | | Group by columns |
|  **GET** | [/api/v1/tableName/distribution/:columnName](#distribution) | |  Distribute data based on column |
|  **GET** | [/api/v1/tableName/distinct/:columnName](#distinct) | |  Find distinct column values |
|  **GET** | [/api/v1/tableName/aggregate/:columnName](#aggregate) | |  Do aggregation on columns |
|  **GET** | [/api/v1/tableName/count](#count) | [where](#query-params) |  Get total rows count |
|  **POST** | [/api/v1/tableName/bulk](#bulk-insert) |  |  Bulk row insert |
|  **PUT** | [/api/v1/tableName/bulk](#bulk-update) |  |  Bulk row update |
|  **DELETE** | [/api/v1/tableName/bulk](#bulk-delete) |  |  Bulk row delete |


### HasMany APIs

|  **Method**  |  **Path**  | **Query Params** | **Description**  |
|---|---|---|---|
|  **GET** | [/api/v1/tableName/has/childTableName](#with-children)  | [where, limit, offset, sort, fields, fields1](#query-params) | List rows of the table with children |
|  **GET** | [/api/v1/tableName/:parentId/childTableName](#children-of-parent)  | [where, limit, offset, sort, fields, fields1](#query-params) | Get children under a certain parent |
|  **POST** | [/api/v1/tableName/:parentId/childTableName](#insert-to-child-table)  |  | Insert children under a certain parent |
|  **GET** | [/api/v1/tableName/:parentId/childTableName/findOne](#findone-under-parent)  | where, limit, offset, sort, fields | Find children under a parent with conditions |
|  **GET** | [/api/v1/tableName/:parentId/childTableName/count](#child-count)  |  | Find children count |
|  **GET** | [/api/v1/tableName/:parentId/childTableName/:id](#get-child-by-primary-key)  |  | Find child by id |
|  **PUT** | [/api/v1/tableName/:parentId/childTableName/:id](#update-child-by-primary-key)  |  | Update child by id |



### BelongsTo APIs

|  **Method**  |  **Path**  | **Query Params** | **Description**  |
|---|---|---|---|
|  **GET** | [/api/v1/childTableName/belongs/parentTablename](#with-parent)  | where, limit, offset, sort, fields | List rows of the table with parent |



## Query params


|  **Name**    | **Alias** | **Use case** | **Default value**  |**Example value**  |
|---|---|---|---|---|
|  [where](#comparison-operators)  | [w](#comparison-operators)  |  Complicated where conditions | | `(colName,eq,colValue)~or(colName2,gt,colValue2)` <br />[Usage: Comparison operators](#comparison-operators) <br />[Usage: Logical operators](#logical-operators) |
|  limit  | l |  Number of rows to get(SQL limit value)  | 10  | 20 |
|  offset  | o |  Offset for pagination(SQL offset value)  | 0  | 20 |
|  sort  | s |  Sort by column name, Use `-` as prefix for descending sort  |   | column_name |
|  fields  | f |  Required column names in result  |  * | column_name1,column_name2 |
|  fields1  | f1 |  Required column names in child result  |  * | column_name1,column_name2 |


#### Comparison operators

```
eq      -   '='         -  (colName,eq,colValue)
not     -   '!='        -  (colName,ne,colValue)
gt      -   '>'         -  (colName,gt,colValue)
ge      -   '>='        -  (colName,ge,colValue)
lt      -   '<'         -  (colName,lt,colValue)
le      -   '<='        -  (colName,le,colValue)
is      -   'is'        -  (colName,is,true/false/null)
isnot   -   'is not'    -  (colName,isnot,true/false/null)
in      -   'in'        -  (colName,in,val1,val2,val3,val4)
btw     -   'between'   -  (colName,btw,val1,val2) 
nbtw    -  'not between'-  (colName,nbtw,val1,val2) 
like    -   'like'      -  (colName,like,%name)
```

#### Example use of comparison operators - complex example
```
/api/payments?where=(checkNumber,eq,JM555205)~or((amount,gt,200)~and(amount,lt,2000))
```

#### Logical operators
```
~or     -   'or'
~and    -   'and'
~not    -   'not'
```

# Examples

### List

##### Request
 ```GET  /api/v1/country```

##### Response
```

    [
        {
            "country_id": 1,
            "country": "Afghanistan",
            "last_update": "2006-02-14T23:14:00.000Z"
        }
    ]
```
    
#### List + where  

##### Request  
``` GET  /api/v1/country?where=(country,like,United%)```
[Usage : comparison operators](#comparison-operators)

##### Response
```
[
    {
        country_id: 101,
        country: "United Arab Emirates",
        last_update: "2006-02-15T04:44:00.000Z"
    },
    {
        country_id: 102,
        country: "United Kingdom",
        last_update: "2006-02-15T04:44:00.000Z"
    },
    {
        country_id: 103,
        country: "United States",
        last_update: "2006-02-15T04:44:00.000Z"
    }
]
```

#### List + where + sort

##### Request
``` GET  /api/v1/country?where=(country,like,United%)&sort=-country```

##### Response
```
[
    {
        country_id: 103,
        country: "United States",
        last_update: "2006-02-15T04:44:00.000Z"
    },
    {
        country_id: 102,
        country: "United Kingdom",
        last_update: "2006-02-15T04:44:00.000Z"
    },
    {
        country_id: 101,
        country: "United Arab Emirates",
        last_update: "2006-02-15T04:44:00.000Z"
    }
]
```

#### List + where + sort + offset

##### Request
``` GET  /api/v1/country?where=(country,like,United%)&sort=-country&offset=1```

##### Response
```
[
    {
        country_id: 102,
        country: "United Kingdom",
        last_update: "2006-02-15T04:44:00.000Z"
    },
    {
        country_id: 101,
        country: "United Arab Emirates",
        last_update: "2006-02-15T04:44:00.000Z"
    }
]
```
#### List + limit

##### Request
``` GET  /api/v1/country?limit=6```

##### Response
```
[
    {
        "country_id": 1,
        "country": "Afghanistan",
        "last_update": "2006-02-14T23:14:00.000Z"
    },
    {
        "country_id": 2,
        "country": "Algeria",
        "last_update": "2006-02-14T23:14:00.000Z"
    },
    {
        "country_id": 3,
        "country": "American Samoa",
        "last_update": "2006-02-14T23:14:00.000Z"
    },
    {
        "country_id": 4,
        "country": "Angola",
        "last_update": "2006-02-14T23:14:00.000Z"
    },
    {
        "country_id": 5,
        "country": "Anguilla",
        "last_update": "2006-02-14T23:14:00.000Z"
    },
    {
        "country_id": 6,
        "country": "Argentina",
        "last_update": "2006-02-14T23:14:00.000Z"
    }
]
```
    
[⤴️](#api-overview)

  
### Get By Primary Key

##### Request  
`GET  /api/v1/country/1`

##### Response
```
    {
        "country_id": 1,
        "country": "Afghanistan",
        "last_update": "2006-02-14T23:14:00.000Z"
    }
```
[⤴️](#api-overview)


### Create

##### Request
```
POST  /api/v1/country
    {
        "country": "Afghanistan"
    }
```

##### Response
```
    {
        "country_id": 1,
        "country": "Afghanistan",
        "last_update": "2006-02-14T23:14:00.000Z"
    }
```
[⤴️](#api-overview)


### Update

##### Request
```
PUT  /api/v1/country/1

    {
        "country": "Afghanistan1"
    }
```

##### Response
```
    {
        "country_id": 1,
        "country": "Afghanistan1",
        "last_update": "20020-02-14T23:14:00.000Z"
    }
```

[⤴️](#api-overview)


### Exists

##### Request
```DELETE  /api/v1/country/1/exists```

#### Response
```
    true
```
[⤴️](#api-overview)


### Delete

##### Request
```DELETE  /api/v1/country/1```

##### Response
```
    1
```



[⤴️](#api-overview)

  
### Find One

##### Request
```GET  /api/v1/country/findOne?where=(country_id,eq,1)```

##### Response
```

    {
        "country_id": 1,
        "country": "Afghanistan",
        "last_update": "2006-02-14T23:14:00.000Z"
    }
```

[⤴️](#api-overview)

  
### Group By

##### Request  
```GET  /api/v1/country/groupby/last_update```

##### Response
```

    [
        {
            "count": 109,
            "last_update": "2006-02-14T23:14:00.000Z"
        },
        {
            "count": 1,
            "last_update": "2020-01-06T15:18:13.000Z"
        },
        {
            "count": 1,
            "last_update": "2020-01-06T14:33:21.000Z"
        }
    ]
 ```   

[⤴️](#api-overview)

  
### Distribution

##### Request  
```GET  /api/v1/payment/distribution/amount```

##### Response
```
    [
        {
            "count": 8302,
            "range": "0-4"
        },
        {
            "count": 3100,
            "range": "5-8"
        },
        {
            "count": 371,
            "range": "9-11.99"
        }
    ]
```
[⤴️](#api-overview)

  
### Distinct

##### Request  
```GET  /api/v1/country/distinct/last_update```

##### Response
```
    [
        {
            "last_update": "2006-02-14T23:14:00.000Z"
        },
        {
            "last_update": "2020-01-06T15:18:13.000Z"
        },
        {
            "last_update": "2020-01-06T14:33:21.000Z"
        },
        {
            "last_update": "2020-01-07T13:42:01.000Z"
        }
    ]
```

[⤴️](#api-overview)

  
### Aggregate

##### Request  
```GET  /api/v1/payment/aggregate/amount?func=min,max,avg,sum,count```

##### Response
```
    [
        {
            "min": 0,
            "max": 11.99,
            "avg": 4.200743,
            "sum": 67413.52,
            "count": 16048
        }
    ]
```

[⤴️](#api-overview)

  
### Count

##### Request  
```GET  /api/v1/country/count```

##### Response
```

    {
        "count": 161
    }
```
[⤴️](#api-overview)

  
### Bulk Insert

##### Request  
```POST  /api/v1/country/bulk
    
    [
        {
            "country": "test 1"
        },
        {
            "country": "test 2"
        }
    ]
```

##### Response
```
    [
        10262
    ]
```
[⤴️](#api-overview)

  
### Bulk Update
##### Request  
```PUT  /api/v1/country/bulk

    [
        {
            "country_id" : 10261,
            "country": "test 3"
        },
        {
            "country_id" : 10262,
            "country": "test 4"
        }
    ]

```
##### Response
```
    [
        1,
        1
    ]
```
[⤴️](#api-overview)

  
### Bulk Delete

##### Request  
```DELETE  /api/v1/country/bulk
    [
        {
            "country_id" : 10261
        },
        {
            "country_id" : 10262
        }
    ]

```
##### Response
```
    [
        1,
        1
    ]
```
[⤴️](#api-overview)

  
### With Children

##### Request  
```GET  /api/v1/country/has/city```

##### Response
```
    [
        {
            "country_id": 1,
            "country": "Afghanistan",
            "last_update": "2006-02-14T23:14:00.000Z",
            "city": [
                {
                    "city_id": 251,
                    "city": "Kabul",
                    "country_id": 1,
                    "last_update": "2006-02-14T23:15:25.000Z"
                },
                ...
            ]
        }
    ]

```

[⤴️](#hasmany-apis)

### Children of parent

##### Request  
```GET  /api/v1/country/1/city```

##### Response
```
    
    [
        {
            "city_id": 251,
            "city": "Kabul",
            "country_id": 1,
            "last_update": "2006-02-14T23:15:25.000Z"
        }
    ]
  ```  

[⤴️](#hasmany-apis)
    
### Insert to child table

##### Request  
```POST  /api/v1/country/1/city

    {
        "city": "test"
    }
```

##### Response
```
    
    {
        "city": "test",
        "country_id": "1",
        "city_id": 10000
    }
```

[⤴️](#hasmany-apis)

### Findone under parent

##### Request
```GET  /api/v1/country/1/city/findOne?where=(city,like,ka%)```

##### Response
```
    
    {
        "city": "test",
        "country_id": "1",
        "city_id": 10000
    }
```


[⤴️](#hasmany-apis)

### Child count

##### Request
```GET  /api/v1/country/1/city/count```

##### Response
```
    {
        "count": 37
    }
```


[⤴️](#hasmany-apis)

### Get Child By Primary key

##### Request
```GET  /api/v1/country/1/city/251```

##### Response
```
    [
        {
            "city_id": 251,
            "city": "Kabul",
            "country_id": 1,
            "last_update": "2006-02-14T23:15:25.000Z"
        }
    ]
```


[⤴️](#hasmany-apis)

### Update Child By Primary key

##### Request
```POST  /api/v1/country/1/city/251
    {
        "city": "Kabul-1"
    }
```

##### Response
```
    1
```


[⤴️](#hasmany-apis)

### Get parent and chlidren within

##### Request
```GET  /api/v1/country/has/city```

##### Response
```
    [
        {
            "city_id": 1,
            "city": "sdsdsdsd",
            "country_id": 87,
            "last_update": "2020-01-02T14:50:49.000Z",
            "country": {
                "country_id": 87,
                "country": "Spain",
                "last_update": "2006-02-14T23:14:00.000Z"
            }
        }
        ...
    ]
```


[⤴️](#belongsto-apis)
### Get table and parent class within

##### Request
```GET  /api/v1/city/belongs/country```

##### Response
```

[
    {
        city_id: 1,
        city: "A Corua (La Corua)",
        country_id: 87,
        last_update: "2006-02-15T04:45:25.000Z",
        country: {
        country_id: 87,
        country: "Spain",
        last_update: "2006-02-15T04:44:00.000Z"
    }
},
```
