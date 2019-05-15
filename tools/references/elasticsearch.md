# ElasticSearch

Unless otherwise specified, all links below reference the [Elasticsearch reference documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html).

## Beginner

If you're not familiar with ElasticSearch, or `Lucene` based search engines in general, you should start here.

The main focus is to be able to query/search after data and fetching documents. Below are some suggestions to where
you might want to start your learning experience:

* [Gettings started](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)
* [Document APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html)
* [Search APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html)
* [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

## Intermediate

The main focus is inserting and updating documents into the indices.

* [Indices APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices.html)
* [Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)

* [elastic4s, client](https://github.com/sksamuel/elastic4s): This is the library we're using to interact
  with elasticsearch from scala. It provides a type safe Scala API for communicating with ElasticSearch.
  > **Note,** we're using the HTTP client, _not_ the TCP client.

## Advanced

Focus on optimise the search engine to improve the search results.

* [Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)
* [Analysis](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)
* [Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)