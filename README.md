# ppelasticsearchelasricstackidep

forward port
```
elastic 127.0.0.1 9200 9200
kibana 127.0.0.1 5601 5601
ssh 127.0.0.1 22 22 
```

install
```
sudo apt update && sudo apt install default-jdk
```
refer to 6.x
[elastic deb](http://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)

(but i install 5.x) change 6 to 5
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```

```
systemctl daemon-reload
```
test file
```
wget http://media.sundog-soft.com/es/shakes-mapping.json
wget http://media.sundog-soft.com/es/shakespeare.json
```


```
curl -XPUT localhost:9200/shakespeare --data-binary @shakes-mapping.json
curl -XPUT localhost:9200/shakespeare/_bulk --data-binary @shakespeare.json
```
test
```
curl -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '{"query":{"match_pharse":{"text_entry："to be"}}}'
```

### Create a Mapping for MovieLens
```
curl -XPUT 127.0.0.1:9200/movies -d '{ "mappings" :{"movie":{"_all":{"enabled":false},"properties":{"year":{"type":"date"}}}}}'
```
test
```
curl -XGET 127.0.0.1:9200/movies/_mapping/movie?pretty
```

### Import a Single Movie via JSON REST
```
curl -XPUT localhost:9200/movies/movie/10999 -d '{ "genre":["MAX"],"title":"inter","year":2000}'
curl -XGET localhost:9200/movies/movie/_search?pretty
```
### Insert Many Movies at Once
```
wget http://media.sundog-soft.com/es/movies.json
curl -XPUT localhost:9200/_bulk?pretty --data-binary @movies.json
```
sample entry:
```
{"create":{"_index":"movies","_type":"movie","_id":"123"}}
{"id":"123","title":"","year":1234,"genre":[""]}
```

### Updating Data in Elasticsearch
```
curl -XPOST localhost:9200/movies/movie/10999/_update -d '{"doc":{"title":"new"}}'
```

### Deleting Data in Elasticsearch
```
curl -XGET localhost:9200/movies/_search?q=Dark
```


### Dealing With Concurrently
```
curl -XPUT localhost:9200/movies/movie/10999?version=3 -d '{"genre":["MIN"],"year":2018}'
```

update with retry_on_confilct
```
curl -XPUT localhost:9200/movies/movie/10999/_update?retry_on_conflict -d '{"doc":{"genre":["MIN"],"year":2018}}'
```

### Using Analyzers and Tokenizers
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '{"query":{"match":{"title":"Star Trek"}}}'
```
recreate mapping
```
curl -XPUT 127.0.0.1:9200/movies -d '{ "mappings" :{"movie":{"_all":{"enabled":false},"properties":{"year":{"type":"date"},"genre":{"type":"string","index":"not_analyzed"}}}}'  # must exact match, case sensitive
```

### Data Modeling with Elasticsearch
__requires update__

## 3
### Search
This is not working, must be encrypted. + means boolean `and`
```
curl -XGET "127.0.0.1:9200/movies/movie/_search?q=+year:>2010+title:trek&pretty"
```



### Using JSON Search
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d ' { "query":{"bool":{"must":{"term":{"title":"trek"}},"filter":{"range":{"year":{"gte":2000}}}}}}'
```
