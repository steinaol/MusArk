TBD: List some relevant `curl` or `httpie` commands for querying the ES API for usefull information during development etc...

# Drop indexes
- all `http DELETE http://localhost:9200/_all`
- a given index `http DELETE http://localhost:9200/myindex`

# Count documents
- all documents (including es internal documents): `http http://localhost:9200/_count`
- documents in indices `http http://localhost:9200/myindex1,myindex2/_count`
- documents in index and type `http http://localhost:9200/myindex1/mytype/_count`