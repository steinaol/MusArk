# Setup

## Local setup
In the local environment we're running elasticsearch in a preconfigured docker container. The default docker image from elasticsearch is preconfigured with a security plugin (X-Pack) that requires a payed license. The custom docker image disables it.

## Production setup
Elasticsearch requires some resources to run so in production we're not running them in a docker container. This requires a manuell installation of elasticsearch.

# Indexes

## Index and type mapping
By default elasticsearch does guess the type for a field when it first see it. That will say that if you insert a document that looks like a date it will create a mapping for that field as a data. If you then inset another document where this field isn't a field then the insert will fail. 

The backend has good control over what kind of types each field in inserting into the indices. To prevent the case described above the schema mapping is defined by the backend. There is other ways do do this, for instance maintaining a `JSON` that stored outside the application scope. We have **not** chosen this approach since the backend knows the data best and does expect some kind of structure to work. This is also essential since the application it self does maintain the indices (indexing and reindexing).

The index it actual index is hidden behind an alias. This is to make it possible to do a reindex and keep the old index available until the new index is ready. When the new index is ready we move the alias from the old index to the new and delete the old index.

## Parent and child documents
To make it easier to keep the documents up to date and to prevent duplication of the documents cross index-types we're linking them with a parent document. This can be done when the documents are stored in the same `index` but in another `type`. This also restricts how the schema is defined since the definition by filed names must not conflict between `types`. 

**Note:** This feature isn't supported in version 6. If elastic doens't provide another way to link or join documents then this need to be changed so we duplicate the content between indices.

## Available indexes
Each module has it's own index.

| index name | types                                                                    | implemented   |
|------------+--------------------------------------------------------------------------+:-------------:|
| objects    | sample, collection                                                         | yes           |
| analysis   | analysis, analysisCollection, sample                              | yes           |
| storage    | nodes, move (object/node), control, observation, environment requirement | no            |
| ...        | ...                                                                      | no            |

### Object index
Contains musit objects (`collection`) and samples (`sample`) where samples has a parent relation to the musit object.

### Analysis index
Contains `analysisCollection`, `analysis` and `sample` where `analysis` has a parent relation to `analysisCollection`.

# Akka streams
When indexing, particular when reindexing we're moving a lot of data from one source to another. To not flod the application with data and make it as effective as possible we're using `akka-streams` to manage the data flow. Akka-streams does so by using back pressure to prevent the application to read more data at the time before it's done writing them.

It also gives us a nice api so we can write flows that for instance populate the document with extra data we need to tweak the documents used for searching.

See the [akka streams documentations](https://doc.akka.io/docs/akka/current/scala/stream/index.html) for more details.

# Searching

## Authorisation
We need to restrict the result set from elasticsearch with the same rules that's applied to the SQL database. This is done by wrapping the user search in a custom search in the backend. That will say that the client can't query directly but has to go trough the backend service.

## Results from searching
The response from elasticsearch is just proxied trough the backend. This is done since the API is stable and we don't need to add anything to the documents.

**Note:** The `_source` and `inner_hit` documents differs from the normal response from the endpoints. They are the actual documents that is stored in elasticsearch.

# Debugging and error handling
Some useful queries are listed here: [wiki: how-to/querying-the-elasticsearch-api](https://gitlab.com/MUSIT-Norway/documentation/wikis/how-to/querying-the-elasticSearch-API)

To be able to query elasticsearch you need to ssh into a machine that's in zone 3 where the elasticsearch instances are running.

## Force a reindex
To force an reindex we need to delete the index (not only the type) and restart the backend service. The service will detect that the index is missing and start a reindexing.

# Resources
- [wiki: references/elasticsearch](https://gitlab.com/MUSIT-Norway/documentation/wikis/references/elasticsearch)
- [wiki: how-to/querying-the-elasticsearch-api](https://gitlab.com/MUSIT-Norway/documentation/wikis/how-to/querying-the-elasticSearch-API)
- [Akka streams](https://doc.akka.io/docs/akka/current/scala/stream/index.html)

# Future improvements

## Health check
The health check does just check if we have any documents in `objects` or `analysis` indices. A health check that verifies that the indexed document count is the same as the documents in the database will indicate that something is wrong. Keep in mind that elasticsearch might lag behind on the indexing.

## API and admin page
Elasticsearch provides a lot of stats that we can show. Also custom status like document count for each index/type might be handy.
We can start a reindexing and still keep the current index but we have no place to start it from. This should be a special operation that only a few user can do.

## Index storage facility documents
Index and make them searchable

## Make the indexes upgradable to elasticsearch version 6.x and above
The next version of elasticsearch (version 6) removes support to create indexes with parent-child relationship. If elasic doens't implement some kind of join like feature the indexer need to be rewritten to duplicate the data into the documents that uses the parent-child relationship.

## Aggregation/facets in search result
[Aggregation/facets](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)