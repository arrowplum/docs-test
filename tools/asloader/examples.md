---
title: Aerospike Loader Examples
description: Usage examples for the asloader tool.
---

Each example given below is for a different use case. To try one of the examples, simply copy the data file content to data.dsv, the config file content to config.json and run the loader using following run command:

    ./run_loader -h localhost -c config.json data.dsv

## Example Use Cases
- [With header in data file](#with_header)
- [Without header in data file](#without_header)
- [Static value](#static_value)
- [System time](#system_time)
- [Options usage](#options)

<a name="with_header"></a>
## With header in data file
The following example illustrates using a datafile and config file which contains all supported data type formats. Referring to the data file, the user can create the config file first. The data file may or may not contain column names as header information. If header information exists then it must be present in first line of the data file and the user can use this header for column position mapping. An example of a data file that does not contain a header file is given [below](#without_header).

### Data file content:
| user_location | user_id | last_visited | set_name | age | user_name | user_name_blob | user_rating |
|---|---|---|---|---|---|---|---|
| IND | userid1 | 04/1/2014 | facebook | 20 | X20 | 583230 | 8.1 |
| USA | userid2 | 03/18/2014 | twitter | 27 | X2 | 5832 | 6.4 |
| UK | userid3 | 01/9/2014 | twitter | 21 | X3 | 5833 | 4.3 |
| UK | userid4 | 01/2/2014 | facebook | 16 | X9 | 5839 | 5.9 |
| IND | userid5 | 08/20/2014 | twitter | 37 | X10 | 583130 | 9.3 |


### Config file content:
```
{
  "version" : "2.0",
  "dsv_config": { "delimiter": "," , "n_columns_datafile": 8, "header_exist": true},

  "mappings": [
      {
          "key": {"column_name":"user_id", "type": "string"},
        
          "set": { "column_name":"set_name", "type": "string"},
        
          "bin_list": [
            {"name": "age",
             "value": { "column_name": "age", "type" : "integer"}
            },
            {"name": "location",
             "value": { "column_name": "user_location", "type" : "string"}
            },
            {"name": "name",
             "value": { "column_name": "user_name", "type" : "string"}
            },
            {"name": "name_blob",
             "value": { "column_name": "user_name_blob", "type" : "blob", "dst_type" : "blob", "encoding":"hex"}
            },
            {"name": "recent_visit",
             "value": { "column_name": "last_visited", "type" : "timestamp", "encoding":"MM/dd/yy", "dst_type": "integer"}
            },
            {"name": "rating",
             "value": { "column_name": "user_rating", "type" : "float"}
            }
          ]
      }
  ]
}
```
### Explanation:
- Specify the delimiter (the separator character between two bin values), n_columns_datafile (number of columns in the data file), header_exist (true if a header exists in the data file.)
- Bin_list contains an array of bin mappings. In each bin mapping there are two entries: 1) Aerospike bin name and 2) Value, which is the bin content mapping. If one column mapping is absent in the config file then that column will be skipped while loading.
- Either column_name or column_position can be used to specify the column.
- Native data types integer and string are stored as-is.
    - add the following line to bin_list in the config file for __integer__ data type:
       - {"name": "age", "value": {  "column_name": "age", "type" : "integer"}}
    - add the following line to bin_list in the config file for __string__ data type:
       - {"name": "location", "value": { "column_name": "user_location", "type" : "string"}}
- Data types other than native types:
    - add the following line to bin_list in the config file for __blob__ data type. Data in data file should be in hex format:
       - {"name": "name_blob", "value": {"column_name": "user_name_blob", "type" : "blob", "dst_type" : "blob" , "encoding":"hex"}
    }
    - add the following line to bin_list in the config file for __float__ data type:
       -  {"name": "rating", "value": {"column_name": "user_rating", "type" : "float"}
    }
    - add the following line to bin_list in the config file for __timestamp__ type data and store it as an integer:
      - { "name": "recent_visit", "value": {"column_name": "last_visited", "type" : "timestamp", "encoding":"MM/dd/yy", "dst_type": "integer"}
    }
    - add the following line to bin_list in the config file for __timestamp__ type data and store it as a string:
      - { "name": "recent_visit", "value": {"column_name": "last_visited", "type" : "timestamp", "encoding":"MM/dd/yy", "dst_type": "string"}
    }
- Specify static set name in the config file as follows:
    - "set": "setnameforall",
    
<a name="without_header"></a>
## Without header in data file
The example below illustrates using a datafile in which there is no header information in first line. When header information is not present in data file always use column_position for column mapping.

### Data file content:
| IND | userid1 | 04/1/2014 | facebook | 20 | X20 | 583230 | 8.1 |
|---|---|---|---|---|---|---|---|
| USA | userid2 | 03/18/2014 | twitter | 27 | X2 | 5832 | 6.4 |
| UK | userid3 | 01/9/2014 | twitter | 21 | X3 | 5833 | 4.3 |
| UK | userid4 | 01/2/2014 | facebook | 16 | X9 | 5839 | 5.9 |
| IND | userid5 | 08/20/2014 | twitter | 37 | X10 | 583130 | 9.3 |

### Config file content:
```
{
  "version" : "2.0",
  "dsv_config": { "delimiter": "," , "n_columns_datafile": 8, "header_exist": false},

  "mappings": [
        {
          "key": {"column_position": 2, "type": "string"},
        
          "set": { "column_position": 4 , "type": "string"},
        
          "bin_list": [
            {"name": "age",
             "value": { "column_position": 5, "type" : "integer"}
            },
            {"name": "location",
             "value": { "column_position": 1, "type" : "string"}
            },
            {"name": "name",
             "value": { "column_position": 6, "type" : "string"}
            },
            {"name": "name_blob",
             "value": { "column_position": 7, "type" : "blob", "dst_type" : "blob" , "encoding":"hex"}
            },
            {"name": "recent_visit",
             "value": { "column_position": 3, "type" : "timestamp", "encoding":"MM/dd/yy", "dst_type": "integer"}
            },
            {"name": "rating",
             "value": { "column_position": 8, "type" : "float"}
            }
          ]
        }
    ]
}
```
### Explanation:

-  As there is no header information in data file so "header_exist" should be false.
-  Each column mapping is specified in column_position only.
-  Check [example-1](#with header) for other details.

<a name="static_value"></a>
## Static value
Apart from loading data from a data file, the user can also add extra information for each record. The example given below inserts user name from the data file and also inserts a "load_from" bin with the contents "xyz database".

### Data file content:
| user_location | user_id | last_visited | set_name | age | user_name | user_name_blob | user_rating |
|---|---|---|---|---|---|---|---|
| IND | userid1 | 04/1/2014 | facebook | 20 | X20 | 583230 | 8.1 |
| USA | userid2 | 03/18/2014 | twitter | 27 | X2 | 5832 | 6.4 |
| UK | userid3 | 01/9/2014 | twitter | 21 | X3 | 5833 | 4.3 |
| UK | userid4 | 01/2/2014 | facebook | 16 | X9 | 5839 | 5.9 |
| IND | userid5 | 08/20/2014 | twitter | 37 | X10 | 583130 | 9.3 |

### Config file content:
```
{
  "version" : "2.0",
  "dsv_config": { "delimiter": "," , "n_columns_datafile": 8, "header_exist": true },

  "mappings": [
      {
          "key": { "column_position": 2, "type": "string"},
        
          "set": { "column_position": 4 , "type": "string"},
        
          "bin_list": [
            {
                "name": "name",
                "value": { "column_position": 6, "type" : "string"}
            },
            {
                "name": "load_from",
                "value": "xyz database"
            }
            
          ]
      }
  ]
}
```
### Explanation:
- Extra information load_from xyz database is added to each record. This is called static bin mapping.

<a name="system_time"></a>
## System time
Apart from loading data from the data file, the user can also add the system time of writing for each record. The example given below inserts user name and system time of writing in milliseconds.

### Data file content:
| user_location | user_id | last_visited | set_name | age | user_name | user_name_blob | user_rating |
|---|---|---|---|---|---|---|---|
| IND | userid1 | 04/1/2014 | facebook | 20 | X20 | 583230 | 8.1 |
| USA | userid2 | 03/18/2014 | twitter | 27 | X2 | 5832 | 6.4 |
| UK | userid3 | 01/9/2014 | twitter | 21 | X3 | 5833 | 4.3 |
| UK | userid4 | 01/2/2014 | facebook | 16 | X9 | 5839 | 5.9 |
| IND | userid5 | 08/20/2014 | twitter | 37 | X10 | 583130 | 9.3 |

### Config file content:
```
{
  "version" : "2.0",
  "dsv_config": { "delimiter": "," , "n_columns_datafile": 8, "header_exist": true },

  "key": { "column_position": 2, "type": "string"},

  "set": { "column_position": 4 , "type": "string"},

  "bin_list": [
    {
		"name": "name",
		"value": { "column_position": 6, "type" : "string"}
    },
    {
    	"name": "write_time",
 		"value": { "column_name": "system_time", "type" : "timestamp", "encoding": "MM/dd/yy HH:mm:ss.SSS", "dst_type": "integer"}
    }
  ]
}
```
### Explanation:
- An additional bin, write_time, is added to each record. If encoding contains SSS then write_time will be in milliseconds otherwise, it is in seconds.

<a name="options"></a>
## Option usage


* With all default values, run aerospike loader as follows:

```java

	./run_loader -c ~/pathto/config.json ~/pathto/data.dsv

```

* Use list of data files for loading:

```java

	./run_loader -c ~/pathto/config.json data1.dsv data2.dsv data3.dsv

```

* Use directory name of data files for loading:

```java

	./run_loader -c ~/pathto/config.json data/

```
* Specify time zone of the location from where data dump is taken. Its optional, if source and destination are same.

```java

	./run_loader -c ~/pathto/config.dsv -tz PST data/

```

* Specify write action for existing records:

```java

	./run_loader -c ~/pathto/config.dsv -wa update data/

```

For more details on the various command line options, refer to [Options](/docs/tools/asloader/options.html).
