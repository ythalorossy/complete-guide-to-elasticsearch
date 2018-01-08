# create index
```json
PUT /product?pretty
```
# delete index
```json
DELETE product
```
# indexing document
```json
POST /product/default
{
  "name": "Processing Events with Logstash",
  "instructor" : {
    "firstName": "Bo",
    "lastName" : "Andersen"
  }
}
```

# indexing document with id
```json
POST /product/default/1
{
  "name": "Complete guide to Elasticsearch",
  "instructor" : {
    "firstName": "Bo",
    "lastName" : "Andersen"
  }
}
```

# retrieving document by id
```json
GET /product/default/1
```

# replacing documet
```json
PUT /product/default/1
{
  "name": "Complete guide to Elasticsearch",
  "instructor" : {
    "firstName": "Bo",
    "lastName" : "Andersen"
  },
  "price": 195
}
```

# updating document
```json
POST /product/default/1/_update
{
  "doc" : { "price": 95, "tags" : ["Elasticsearch"]}
}
```

# updating document using scripts
```json
POST product/default/1/_update
{
  "script" : "ctx._source.price += 10"
}
```

# delete document
```json
DELETE /product/default/1
```

# upsert document
```json
POST product/default/1/_update
{
  "script" : "ctx._source.price += 5",
  "upsert" : {
    "price": 100
  }
}
```

# delete index
```json
DELETE /product
```

# bulk
```json
POST /product/default/_bulk
{ "index": { "_id": "100" }}
{ "price": 100 }
{ "index": { "_id": "101" }}
{ "price": 101 }

POST /product/default/_bulk
{ "update" : { "_id": "100" } }
{ "doc" : {"price": 1000 } }
{ "delete" : { "_id": "101"} }
```
```json
GET product/default/100
```
# health
GET _cat/health?v
# nodes
GET _cat/nodes?v
# indices
GET _cat/indices?v
# allocation
GET _cat/allocation?v
# shards
GET _cat/shards?v


# mapping
```json
GET product/default/_mapping
```
```json
PUT product/default/_mapping
{
  "properties": {
    "discount": {
      "type": "double"
    }
  }
}
```

# mapping - types and fields
```json
PUT /product
{
  "mappings": {
    "default": {
      "dynamic": false, 
      "properties": {
        "in_stock" : {
          "type": "integer"
        },
        "is_active" : {
          "type": "boolean"
        },
        "price" : {
          "type": "integer"
        },
        "sold" : {
          "type": "long"
        }
      }
    }
  }
}
```

# mapping - dates
```json
PUT product/default/_mapping
{
  "properties": {
    "created": {
      "type": "date",
      "format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd"
    }
  }
}
```

# mapping - keywords
```json
PUT product/default/_mapping
{
  "properties": {
    "description" : {
      "type": "text"
    },
    "name": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      }
    },
    "tags": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      }
    }
  }
}
```

# analyzers
```json
POST _analyze
{
  "tokenizer": "standard",
  "text": "I'm in the mood for drinking semi-dry red wine!"
}

POST _analyze
{
  "filter": ["lowercase"],
  "text": "I'm in the mood for drinking semi-dry red wine!"
}

POST _analyze
{
  "analyzer": "standard",
  "text": "I'm in the mood for drinking semi-dry red wine!"
}

PUT /existing_analyzer_config
{
	"settings": {
	  "analysis": {
	    "analyzer": {
	      "english_stop": {
	        "type" : "standard",
	        "stopwords": "_english_"
	      }
	    },
	    "filter": {
	      "my_stemmer": {
          "type": "stemmer",
	        "name": "english"
	      }
	    }
	  }
	}
}

POST /existing_analyzer_config/_analyze
{
  "analyzer" : "english_stop",
  "text" : "I'm in the mood for drinking semi-dry red wine!"
}

POST /existing_analyzer_config/_analyze
{
  "tokenizer" : "standard",
  "filter": [ "my_stemmer" ],
  "text" : "I'm in the mood for drinking semi-dry red wine!"
}

PUT /existing_analyzer_config
{
	"settings": {
	  "analysis": {
	    "analyzer": {
	      "english_stop": {
	        "type" : "standard",
	        "stopwords": "_english_"
	      }
	    },
	    "filter": {
	      "my_stemmer": {
          "type": "stemmer",
	        "name": "english"
	      }
	    }
	  }
	}
}

POST /existing_analyzer_config/_analyze
{
  "analyzer" : "english_stop",
  "text" : "I'm in the mood for drinking semi-dry red wine!"
}

POST /existing_analyzer_config/_analyze
{
  "tokenizer" : "standard",
  "filter": [ "my_stemmer" ],
  "text" : "I'm in the mood for drinking semi-dry red wine!"
}

DELETE /analyzers_test

PUT /analyzers_test
{
	"settings": {
	  "analysis": {
	    "analyzer": {
	      "english_stop": {
	        "type" : "standard",
	        "stopwords": "_english_"
	      },
	      "my_analyzer" : {
          "type": "custom",
	        "tokenizer": "standard",
	        "char_filter": [
	          "html_strip"
          ],
          "filter": [
            "standard",
            "lowercase",
            "trim",
            "my_stemmer"
          ]	         
	      }
	    },
	    "filter": {
	      "my_stemmer": {
          "type": "stemmer",
	        "name": "english"
	      }
	    }
	  }
	}
}

POST /analyzers_test/_analyze
{
  "analyzer" : "my_analyzer",
  "text" : "I'm in the mood for drinking <strong>semi-dry</strong> red wine!"
}

PUT analyzers_test/default/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_analyzer"
    },
    "teaser" : {
      "type": "text",
      "analyzer": "standard"
    }
  }
}

POST analyzers_test/default/1
{
  "description": "drinking",
  "teaser": "drinking"
}

GET analyzers_test/default/_search
{
  "query" : {
    "term": {
      "teaser": {
        "value" : "drinking"
      }
    }
  }
}
```

# no result - using stemmer drinking to drink
```json
GET analyzers_test/default/_search
{
  "query" : {
    "term": {
      "description": {
        "value" : "drinking"
      }
    }
  }
}
```

# Dynamic index - changing settings

```json
POST analyzers_test/_close

PUT analyzers_test/_settings
{
  "analysis": {
    "analyzer": {
      "french_stop": {
        "type": "standard",
        "stopwords": "_french_"
      }
    }
  }
}

POST analyzers_test/_open
```

# search - request URI
```json
GET product/default/_search?q=*
GET product/default/_search?q=name:Lobster
GET product/default/_search?q=tags:Meat AND name:Tuna
```

# search match_all
```json
GET product/default/_search
{
  "query": {
    "match_all": {}
  }
}
```

# search explain
```json
GET product/default/_search?explain
{
  "query": {
    "term": {
      "name": {
        "value": "lobster"
      }
    }
  }
}
```

# search debug
```json
GET product/default/1/_explain
{
  "query" : {
    "term": {
      "name": {
        "value": "lobster"
      }
    }
  }
}
```

# search using term (not analyzed)
```json
GET product/default/_search
{
  "query": {
    "term": {
      "is_active": {
        "value": true
      }
    }
  }
}
```

# search multiple terms
```json
GET product/default/_search
{
  "query": {
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}
```

# retrive documents by IDs
```json
GET product/default/_search
{
  "query": {
    "ids": {
      "values": [1, 2, 3]
    }
  }
}
```

# retrieve documets with range (value)
```json
GET product/default/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 1,
        "lte": 5
      }
    }
  }
}
```

# retrieve documets with range (date)
```json
GET product/default/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01-01-2010",
        "lte": "31-12-2010",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
```

# retrieve documets with range (relative date)
```json
GET product/default/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y"
      }
    }
  }
}

GET product/default/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now-1M+18d"
      }
    }
  }
}
```

# retrieve document with non-null values
```json
GET product/default/_search
{
  "query": {
    "exists" : {
      "field" : "tags"
    }
  }
}
```

# retrieve document matching on prefixed
```json
GET product/default/_search
{
  "query": {
    "prefix": {
      "tags.keyword": {
        "value": "Vege"
      }
    }
  }
}
```

# searching using wildcard
```json
GET product/default/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": {
        "value": "Veg*ble"
      }
    }
  }
}
```

# searching using regular expressions
```json
GET product/default/_search
{
  "query": {
    "regexp": {
      "tags.keyword": {
        "value": "Veg[a-zA-Z]+ble"
      }
    }
  }
}
```

## Lesson 01
```json
GET product/default/_search
{
  "query": {
    "range": {
      "sold": {
        "lt": 10
      }
    }
  }
}

GET product/default/_search
{
  "query": {
    "range": {
      "sold": {
        "gte": 10,
        "lt": 30
      }
    }
  }
}

GET product/default/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "Meat"
      }
    }
  }
}

GET product/default/_search
{
  "query": {
    "terms": {
      "name": [
        "Tomato",
        "Paste"
      ]
    }
  }
}

GET product/default/_search
{
  "query": {
    "wildcard": {
      "name": {
        "value": "past*"
      }
    }
  }
}

GET product/default/_search
{
  "query": {
    "regexp": {
      "name": {
        "value": "[0-9]+"
      }
    }
  }
}

GET recipe/default/_mapping
```

# matching with match query (operators OR AND)
```json
GET recipe/default/_search
{
  "query": {
    "match": {
      "title": "Recipes with pasta or spaghetti"
    }
  }
}

GET recipe/default/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Recipes with pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}

GET recipe/default/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}
```

# matching phrases - The order of the values is important
```json
GET recipe/default/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
```

# searching multi fields
```json
GET recipe/default/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": ["title", "description"]
    }
  }
}
```

## Lesson 02
```json
GET recipe/default/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta with parmesan and spinach"
      }
    }
  }
}

GET recipe/default/_search
{
  "query": {
    "match": {
      "title": {
      "query": "pasta carbonara",
      "operator": "and"
      }
    }
  }
}

GET recipe/default/_search
{
  "query": {
    "multi_match": {
      "query": "pasta pesto",
      "fields": ["title", "description"]
    }
  }
}
```

# searchig bool logic - must
```json
GET recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

# searchig bool logic - must and filter
```json
GET recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }  
      ]
    }
  }
}
```

# searchig bool logic - must, must_not and filter
```json
GET recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ], 
      "should": [
        {
          "match": {
            "ingredients.name": "parsley"
          }
        }
      ], 
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }  
      ]
    }
  }
}
```

# searchig bool logic - should
```json
GET recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "pasta"
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ]
    }
  }
}
```

# searchig bool logic - named queries
```json
GET recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": {
              "query": "parmesan",
              "_name": "parmesan_must"
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": {
              "query": "tuna",
              "_name": "tuna_must_not"
            }
          }
        }
      ], 
      "should": [
        {
          "match": {
            "ingredients.name": {
              "query": "parsley",
              "_name": "parsley_should"
            }
          }
        }
      ], 
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15,
              "_name": "prep_time_filter"
            }
          }
        }  
      ]
    }
  }
}
```

### MUST similar to AND
### SHOULD similar to OR

# nested objects
```json
PUT /department
{
  "mappings": {
    "default": {
      "properties": {
        "name": {
          "type": "text"
        },
        "employees": {
          "type": "nested"
        }
      }
    }
  }
}
```
```json
POST /department/default/1
{
  "name": "Development",
  "employees": [
    {
      "name": "Eric Green",
      "age": 39,
      "gender": "M",
      "position": "Big Data Specialist"
    },
    {
      "name": "James Taylor",
      "age": 27,
      "gender": "M",
      "position": "Software Developer"
    },
    {
      "name": "Gary Jenkins",
      "age": 21,
      "gender": "M",
      "position": "Intern"
    },
    {
      "name": "Julie Powell",
      "age": 26,
      "gender": "F",
      "position": "Intern"
    },
    {
      "name": "Benjamin Smith",
      "age": 46,
      "gender": "M",
      "position": "Senior Software Engineer"
    }
  ]
}
```
```json
POST /department/default/2
{
  "name": "HR & Marketing",
  "employees": [
    {
      "name": "Patricia Lewis",
      "age": 42,
      "gender": "F",
      "position": "Senior Marketing Manager"
    },
    {
      "name": "Maria Anderson",
      "age": 56,
      "gender": "F",
      "position": "Head of HR"
    },
    {
      "name": "Margaret Harris",
      "age": 19,
      "gender": "F",
      "position": "Intern"
    },
    {
      "name": "Ryan Nelson",
      "age": 31,
      "gender": "M",
      "position": "Marketing Manager"
    },
    {
      "name": "Kathy Williams",
      "age": 49,
      "gender": "F",
      "position": "Senior Marketing Manager"
    },
    {
      "name": "Jacqueline Hill",
      "age": 28,
      "gender": "F",
      "position": "Junior Marketing Manager"
    },
    {
      "name": "Donald Morris",
      "age": 39,
      "gender": "M",
      "position": "SEO Specialist"
    },
    {
      "name": "Evelyn Henderson",
      "age": 24,
      "gender": "F",
      "position": "Intern"
    },
    {
      "name": "Earl Moore",
      "age": 21,
      "gender": "M",
      "position": "Junior SEO Specialist"
    },
    {
      "name": "Phillip Sanchez",
      "age": 35,
      "gender": "M",
      "position": "SEM Specialist"
    }
  ]
}
```

# this query don't work with nested objects 
```json
GET department/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "employees.position": "intern"
          }
        },
        {
          "term": {
            "employees.gender.keyword": {
              "value": "F"
            }
          }
        }
      ]
    }
  }
}
```

# search using nested queries
```json
GET department/default/_search
{
  "query": {
    "nested": {
      "path": "employees",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employees.position": "intern"
              }
            },
            {
              "term": {
                "employees.gender.keyword": {
                  "value": "F"
                }
              }
            }
          ]
        }        
      }
    }
  }
}
```

# source filtering
```json
GET recipe/default/_search
{
  "_source": "ingredients.*", 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

# filter
```json
GET recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "pasta"
          }
        }
      ],
      "filter": [
        {
          "range" : {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

# orders
```json
PUT /order
{
  "mappings": {
    "default": {
      "properties": {
        "purchased_at": {
          "type": "date"
        },
        "lines": {
          "type": "nested",
          "properties": {
            "product_id": {
              "type": "integer"
            },
            "amount": {
              "type": "double"
            },
            "quantity": {
              "type": "short"
            }
          }
        },
        "total_amount": {
          "type": "double"
        },
        "status": {
          "type": "keyword"
        },
        "sales_channel": {
          "type": "keyword"
        },
        "salesman": {
          "type": "object",
          "properties": {
            "id": {
              "type": "integer"
            },
            "name": {
              "type": "text"
            }
          }
        }
      }
    }
  }
}

GET /order/_mapping

GET order/default/_search
```

# metric aggregation
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "total_amount"
      }
    },
    "avg_sale": {
      "avg": {
        "field": "total_amount"
      }
    },
    "min_sale": {
      "min": {
        "field": "total_amount"
      }
    },
    "max_sale": {
      "max": {
        "field": "total_amount"
      }
    }
  }
}
```

# metric aggregation cardinality (distinct values)
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "cardinality": {
      "cardinality": {
        "field": "salesman.id"
      }
    }
  }
}
```

### Query with aggregation.
### The aggregation sum the amount of documents used
```json
GET order/default/_search
{
  "_source": "total_amount", 
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "total_amount": {
              "gte": 250
            }
          }
        }
      ]
    }
  }, 
  "aggs": {
    "values_count" : {
      "value_count": {
        "field": "total_amount"
      }
    }
  }
}
```

# metric aggregation stats
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "amount_stats": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}
```

# buckets aggregation
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "missing": "N/A",
        "min_doc_count": 0,
        "order": {
          "_term": "asc"
        }
      }
    }
  }
}
```

# buckets aggregation
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "total_amount",
        "size": 10
      }
    }
  }
}
```

# buckets aggregation - nested
```json
GET order/default/_search
{
  "size": 0,
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  }, 
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 10
      },
      "aggs": {
        "NAME": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

# buckets aggregation nested - filter
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "low_value" : {
      "filter": {
        "range": {
          "total_amount": {
            "lt": 50
          }
        }
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

# buckets aggregation filter rules
```json
GET recipe/default/_search
{
  "size": 0,
  "aggs": {
    "my_filters" : {
      "filters": {
        "filters": {
          "pasta": {
            "match" : {
              "title" : "pasta"
            }
          },
          "spaghetti": {
            "match" : {
              "title" : "spaghetti"
            }
          }
        }
      },
      "aggs": {
        "avg_ratings": {
          "avg": {
            "field": "ratings"
          }
        }
      }
    }
  }
}
```

# aggregation range values
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "amount_distribuition": {
      "range": {
        "field": "total_amount",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}
```

# aggregation - range dates
```json
GET order/default/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd",
        "keyed": true, 
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M",
            "key": "first_half"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y",
            "key": "second_half"
          }
        ]
      },
      "aggs": {
        "bucket_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

# aggregation histograms intervals
```json
GET order/default/_search
{
  "size": 0, 
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  }, 
  "aggs": {
    "amount_distibuition": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 0,
        "extended_bounds": {
          "min": 0,
          "max": 500
        }
      }
    }
  }
}
```

# aggregation histograms date
```json
GET order/default/_search
{
  "size": 0, 
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  }, 
  "aggs": {
    "orders_over_time": {
      "date_histogram": {
        "keyed": true,
        
        "format": "yyyy-MM-dd", 
        "field": "purchased_at",
        "interval": "month",
        "min_doc_count": 0
      }
    }
  }
}
```

# aggregation global
```json
GET order/default/_search
{
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "size": 0,
  "aggs": {
    "stats_expensive": {
      "stats": {
        "field": "total_amount"
      }
    },
    "all_orders": {
      "global": {},
      "aggs": {
        "stats_amount": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}

POST order/default/1001
{
  "total_amount": 100
}

POST order/default/1002
{
  "total_amount": 200,
  "status": null
}

GET order/default/_search
{
  "size": 0,
  "aggs": {
    "orders_without_status": {
      "missing": {
        "field": "status"
      },
      "aggs": {
        "missing_sum": {
          "sum": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}

DELETE order/default/1001
DELETE order/default/1002

PUT /proximity/default/1
{
  "title": "Spicy Sauce"
}

PUT /proximity/default/2
{
  "title": "Spicy Tomato Sauce"
}

PUT /proximity/default/3
{
  "title": "Spicy Tomato and Garlic Sauce"
}

PUT /proximity/default/4
{
  "title": "Tomato Sauce (spicy)"
}

PUT /proximity/default/5
{
  "title": "Spicy and very delicious Tomato Sauce"
}
```

# search proximity
```json
GET /proximity/default/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "spicy sauce",
        "slop": 2
      }
    }
  }
}
```

# relevance score with proximity
```json
GET /proximity/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": {
              "query": "spicy sauce"
            }
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": {
              "query": "spicy sauce",
              "slop": 5
            }
          }
        }
      ]
    }
  }
}
```

# fuzzness
### max fuzzness MUST be 2
```json
GET /product/default/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bst3r",
        "fuzziness": "auto"
      }
    }
  }
}

GET /product/default/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster love",
        "operator": "and", 
        "fuzziness": 1
      }
    }
  }
}
```

# fuzzness + transposition
### AB <-> BA
### lvie <-> live
```json
GET /product/default/_search
{
  "query": {
    "match": {
      "name": {
        "query": "lvie",
        "fuzziness": 1,
        "fuzzy_transpositions": false
      }
    }
  }
}
```

# fuzz query
### not analyzed
```json
GET /product/default/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "LOBSTER",
        "fuzziness": "auto"
      }
    }
  }
}

GET /product/default/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "lobster",
        "fuzziness": "auto"
      }
    }
  }
}
```

# synonyms
```json
PUT /synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym", 
          "synonyms": [
            "awful => terrible",
            "awesome => great, super",
            "elasticsearch, logstash, kibana => elk",
            "weird, strange"
          ]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_test"
          ]
        }
      }
    }
  },
  "mappings": {
    "default": {
      "properties": {
        "description": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
}

POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "awesome"
}

POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Elasticsearch"
}

POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "position1 weird"
}

POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Elasticsearch is awesome, but can also seem weird sometimes."
}

POST /synonyms/default
{
  "description": "Elasticsearch is awesome, but can also seem weird sometimes."
}

GET /synonyms/default/_search
{
  "query": {
    "match": {
      "description": "elk"
    }
  }
}
```

# highlights
```json
POST /highlighting/default/1
{
  "description": "Let me tell you a story about Elasticsearch. It's a full-text search engine that is built on Apache Lucene. It's really easy to use, but also packs lots of advanced features that you can use to tweak its searching capabilities. Lots of well-known and established companies use Elasticsearch, and so should you!"
}

GET /highlighting/default/_search
{
  "_source": false,
  "query": {
    "match": { "description": "Elasticsearch story" }
  },
  "highlight": {
    "fields": {
      "description" : {}
    }
  }
}
```

# stemming
```json
PUT /stemming_test
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym",
          "synonyms": [
            "firm => company",
            "love, enjoy"
          ]
        },
        "stemmer_test" : {
          "type" : "stemmer",
          "name" : "english"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_test",
            "stemmer_test"
          ]
        }
      }
    }
  },
  "mappings": {
    "default": {
      "properties": {
        "description": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
}

POST /stemming_test/default/1
{
  "description": "I love working for my firm!"
}

GET /stemming_test/default/_search
{
  "query": {
    "match": {
      "description": "enjoy work"
    }
  },
  "highlight": {
    "fields": {
      "description" : {}
    }
  }
}
```
