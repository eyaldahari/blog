# Reindexing documents in Elasticsearch

It is very easy to start working with Elasticsearch.
The fact that Elasticsearch do not require a schema upfront indexing your data is an advantage.
However, this sometimes comes with a price. Although you can add new types to an index, or add new fields to a type, you
can't add new analyzers or make changes to existing fields' mappings. For instance, if a field was mapped as a `string` you can't change its type to `long` after data was indexed. If you were to do so, the data that had already been indexed would be incorrect and your
searches would no longer work as expected.

How to solve this problem? Reindex!  
Elasticsearch 2.3 was realeased with a new Reindex API which I am going to cover here in this post.

In order to illustrate re-indexing, throughout this post I'll use an Elasticsearch index named _library_ and _books_ as Elasticsearch type.

Now lets look for example at the following _book_ document:

```sh
curl -XPUT 'http://localhost:9200/library/books/1' -d '
{
    "title": "Crime and Punishment",
    "price": 9
}'
```

Indexing the document above would automatically create the following mappings in Elasticsearch:

```sh
# Get mappings:
curl -XGET 'http://localhost:9200/library/books/_mapping'

# Returns:
{
   "library": {
      "mappings": {
         "books": {
            "properties": {
               "price": {
                  "type": "long"
               },
               "title": {
                  "type": "string"
               }
            }
         }
      }
   }
}
```

Now, say for example that I want to change the type of _price_ from _long_ to _float_.
This is not possible after I have indexed my _"Crime and Punishment"_.

Lets try and see what happens:
```sh
# Update mapping request:
curl -XPOST 'http://localhost:9200/library/_mapping/books' -d '
{
  "properties": {
    "price": {
      "type": "float"
    },
    "title": {
      "type": "string"
    }
  }
}'

# Returns an error:
{
   "error": {
      "root_cause": [
         {
            "type": "illegal_argument_exception",
            "reason": "mapper [price] of different type, current_type [long], merged_type [float]"
         }
      ],
      "type": "illegal_argument_exception",
      "reason": "mapper [price] of different type, current_type [long], merged_type [float]"
   },
   "status": 400
}
```

Another way to overcome this situation is to add another field: _accurate_price_ as follows:
```sh
# Update mapping request
curl -XPOST 'http://localhost:9200/library/mapping/books' -d '
{
  "properties": {
    "price": {
      "type": "long"
    },
    "accurate_price": {
      "type": "float"
    },
    "title": {
      "type": "string"
    }
  }
}'

# The request above would add new mapping to books type
curl -XGET 'http://localhost:9200/library/books/_mapping'

# Returns:
{
   "library": {
      "mappings": {
         "books": {
            "properties": {
               "accurate_price": {
                  "type": "float"
               },
               "price": {
                  "type": "long"
               },
               "title": {
                  "type": "string"
               }
            }
         }
      }
   }
}
```

There are situations where you have to reindex. To name a few:
- Add more primary shards to your index in case it grows in capacity more than planned ahead
- Change default analyzers
- Add more complicated mappings to types
- There are more..

The simplest way to apply these changes to your existing data is to
reindex:  create a new index with the new settings and copy all of your
documents from the old index to the new index.

One of the advantages of the `_source` field is that you already have the
whole document available to you in Elasticsearch itself. You don't have to
rebuild your index from the database, which is usually much slower. Elasticsearch uses that for re-indexing

#Start Reindexing
- Create a new index named _new_library_ and add all relevant settings
```json
curl -XPUT 'http://localhost:9200/new_library' -d '
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
    "books": {
      "properties": {
        "price": {
          "type": "float"
        },
        "title": {
          "type": "string",
          "index" : "not_analyzed"
        }
      }
    }
  }
}'

# Result
{
  "new_library": {
    "mappings": {
      "books": {
        "properties": {
          "price": {
            "type": "float"
          },
          "title": {
            "type": "string",
            "index": "not_analyzed"
          }
        }
      }
    }
  }
}
```

- Once you have the _new_library_ index configured you can start re-indexing
```json
curl -XPOST 'http://localhost:9200/_reindex' -d '
{
  "source": {
    "index": "library"
  },
  "dest": {
    "index": "new_library",
    "version_type": "external"
  }
}'

#Result 
{
  "took": 22,
  "timed_out": false,
  "total": 2,
  "updated": 0,
  "created": 2,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": 0,
  "failures": []
}
```

- As you can see, we need to specify the source index (using the source section), the destination index (using the desc section) and send the command to the _reindex REST end-point. We’ve also specified the version_type and set it to external to preserve document versions. 
- The result shows I have reindexed 2 documents, the two where created since they did not exists in the _new_library_ before.
- We can also see a few useful statistics about the re-indexing process:  
    took – the amount of the the re-indexing operation took,  
    updated – number of documents updated,  
    batches – number of batches used,  
    version_conflicts – how many documents were conflicting,  
    failures – information about documents that failed to be reindexed, none in our case,  
    created – number of created documents, which is 18 in our case.   

Please note that there are a lot more settings you can apply on the re-indexing API and control many more configurations as well as re-indexing according to the result of a query, wait for it to finish, get re-indexing stats and etc.

All this information may be found at [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) and a great [Sematext blog](https://sematext.com/blog/2016/03/21/reindexing-data-with-elasticsearch/) I used.


Hope that it helps! :)

Follow me on:

[Medium](https://medium.com/@eyaldahari) | [Twitter](https://twitter.com/EyalDahari) | [Linkedin](https://www.linkedin.com/in/eyaldahari) | [Stackoverflow](http://stackexchange.com/users/7651751/e-dahari?tab=activity) | [GitHub](https://github.com/eyaldahari)
