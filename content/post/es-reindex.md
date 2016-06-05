# Reindexing data in Elasticsearch

It is very easy to start working with Elasticsearch.
The fact that Elasticsearch do not require a schema upfront indexing your data is an advantage.
However, this sometimes comes with a price. Although you can add new types to an index, or add new fields to a type, you
can't add new analyzers or make changes to existing fields' mappings. For instance, if a field was mapped as a `string` you can't change its type
to `long` after data was indexed. If you were to do so, the data that had already been indexed would be incorrect and your
searches would no longer work as expected.

Let's look at some examples.
To illustrate reindexing, throughout this post I'll use _library_ as Elasticsearch index and _books_ as Elasticsearch type.

Now lets take for example the following _book_ document:

```sh
curl -XPUT 'http://localhost:9200/library/books/1' -d '
{
    "title": "Crime and Punishment",
    "price": 9
}'
```

Indexing the document above would automatically create the following mappings in Elasticsearch:

```sh
# Get mapping:
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

Now, say for instance that I want to have the _price_ of the _book_ as a _string_ and not _long_.
This is not possible after I have indexed my _"Crime and Punishment"_.

Lets try and see what happens:
```sh
# Update request:
curl -XPOST 'http://localhost:9200/library/books/1' -d '
{
    "title": "Crime and Punishment",
    "price": "9$"
}'

# Returns an error:
{
   "error": {
      "root_cause": [
         {
            "type": "mapper_parsing_exception",
            "reason": "failed to parse [price]"
         }
      ],
      "type": "mapper_parsing_exception",
      "reason": "failed to parse [price]",
      "caused_by": {
         "type": "number_format_exception",
         "reason": "For input string: \"9$\""
      }
   },
   "status": 400
}
```

Another way to overcome this situation is to add another field: _price_in_us_ as follows:
```sh
# Update request
curl -XPOST 'http://localhost:9200/library/books/1/_update' -d '
{
    "doc":{ "price_in_us": "9$" }
}'

# The request above would add new mapping to books type
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
               "price_in_us": {
                  "type": "string"
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

In order to overcome this situations you'll have to reindex.
This means creating a new index add new settings apply it with new mappings 
and all needed functionality  

The simplest way to apply these changes to your existing data is to
reindex:  create a new index with the new settings and copy all of your
documents from the old index to the new index.

One of the advantages of the `_source` field is that you already have the
whole document available to you in Elasticsearch itself. You don't have to
rebuild your index from the database, which is usually much slower.

To reindex all of the documents from the old index efficiently,  use
<<scroll,_scroll_>> to retrieve batches((("using in reindexing documents"))) of documents from the old index,
and the <<bulk,`bulk` API>> to push them into the new index.

.Reindexing in Batches
****

You can run multiple reindexing jobs at the same time, but you obviously don't
want their results to overlap.  Instead, break a big reindex down into smaller
jobs by filtering on a date or timestamp field:

[source,js]
--------------------------------------------------
GET /old_index/_search?scroll=1m
{
    "query": {
        "range": {
            "date": {
                "gte":  "2014-01-01",
                "lt":   "2014-02-01"
            }
        }
    },
    "sort": ["_doc"],
    "size":  1000
}
--------------------------------------------------


If you continue making changes to the old index, you will want to make
sure that you include the newly added documents in your new index as well.
This can be done by rerunning the reindex process, but again filtering
on a date field to match only documents that have been added since the
last reindex process started.

****

