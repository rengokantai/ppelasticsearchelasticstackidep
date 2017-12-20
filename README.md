# ppelasticsearchelasticstackidep

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
terms
```
term
terms
range
exists
missing
bool
``` 
```
match
match_all
multi_match
```


```
match --- match one word
match_phrase --- exact match
```


### Pagination
may kill performance
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?size=2&from=2&pretty'
```

### Sorting
A string field that is analyzed for full-text search can't be used to sort documents

##### If you need to sort on an analyzed field, map a not_analyzed copy
```
-d
'{
"mappings":"movie":{
  "_all":{"enabled":false},
  "properties":{
    "title":{
      "type":"string",
      "fields":{
        "raw":{
          "type":"string",
          "index":"not_analyzed"
        }
      }
    }
  }
}
}}'
```
when sorting, use `raw` field
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?sort=title.raw&pretty'
```

### Using Filters
```
'
{
  "query":{
    "bool":{
      "must":{"match":{"genre":""}},
      "must_not":{"match":{"title":"trek"}},
      "filter":{"range":{"year":{"gte":2000,"lt":2011}}}
    }
  }
}
'
```

### Fuzzy Queries
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '{ "query":{"fuzzy":{"title":{"value":"worrd",fuzziness:2}}}}'
```

### Partial Matching
prefix, wildcard
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '{ "query":{"wildcard":{"year":19**}}}'
```

### N-Grams, and Search as You Type
##### query-time search as-you-type
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '{ "query":{"match_phrase_prefix":{"title":{"query":"star trek",slop:2}}}}'
```

#### 03:50
index n-grams
```
{
  "settings":{
    "analysis":{
      "filter":{
        "autocomplete_filter":{
          "type":"edge_ngram",
          "min_gram":1,
          "max_gram":10
        }
      },
      "analyzer":{
        "autocomplete":{
          "type":"custom",
          "tokenizer":"standard",
          "filter":[
            "lowercase",
            "autocomplete_filter"
          ]
        }
      }
    }
  }
}
```

```
curl -XGET '127.0.0.1:9200/movies/_analyze?analyzer=autocomplete&pretty' -d 'Sta'
```
#### 11:39 
standard analyzer
```
{"query":{"match":{"title":{"query":"sta","analyzer":"standard"}}}}
```


Importing Data from Scripts
download
```
wget http://files.grouplens.org/datasets/movielens/ml-latest-small.zip
unzip ml-latest-small.zip
wget media.sundog-soft.com/es/MoviesToJson.py
python3 MoviesToJson.py > moremovies.json
```

purge and import
```
curl -XDELETE localhost:9200/movies
curl -XPUT localhost:9200/_bulk --data-binary @moremovies.json
```

```
apt install python3-pip
pip3 install elasticsearch
```
```
wget http://media.sundog-soft.com/es/IndexRatings.py
python3 IndexRatings.py
```

### Installing Logstash
```
vim /etc/logstash/conf.d/logstash.conf   (need to add this file by hand)
```


### Importing Apache Access Logs with Logstash
```
cd /usr/share/logstash
bin/logstash --path.settings=/etc/logstash -t(test)
```
__this syntax is incorrect!!__
```
bin/logstash -f /etc/logstash/conf.d/logstash.conf
```

v means show results as list
```
curl -XGET localhost:9200/_cat/indices?v
curl -XGET localhost:9200/logstash-2017.05.02/_search?pretty'
```


### Import Data from MySQL using Logstash
```
create database moveielens;
create tablemovielens.movies (
movieID int primary key not null,
title text,
releaseDate date
)
load data local infile 'item' into table movielens.movies fields terminated by '|' (movieID,title,@var3) set releaseDate = STR_TO_DATE(@var3,'%d-%M-%Y);
```



### Buckets and Metrics
```
curl -XGET 'localhost:9200/ratings.rating/_search?size=0&pretty' -d '
{
  "aggs":{
    "ratings":{
      "terms":{
        "field":"rating"
      }
    }
  }
}'
```
count only 5-star ratings
```
curl -XGET 'localhost:9200/ratings.rating/_search?size=0?pretty' -d '
{ 
  "query":{
    "match":{
      "rating":5.0
    }
  },
  "aggs":{
    "ratings":{
      "terms":{
        "field":"rating"
      }
    }
  }
}'
```
create a average rating
```
curl -XGET 'localhost:9200/ratings.rating/_search?size=0?pretty' -d '
{ 
  "query":{
    "match_phrase":{
      "title":"star"
    }
  },
  "aggs":{
    "avg_rating":{
      "avg":{
        "field":"rating"
      }
    }
  }
}'
```
