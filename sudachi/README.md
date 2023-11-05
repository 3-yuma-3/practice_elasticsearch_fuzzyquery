# sudachi

## 参考

- [WorksApplications / elasticsearch-sudachi (github)](https://github.com/WorksApplications/elasticsearch-sudachi)
  - [elasticsearch-sudachi/docs/tutorial.md](https://github.com/WorksApplications/elasticsearch-sudachi/blob/develop/docs/tutorial.md)
- [検索基盤チームのElasticsearch×Sudachi移行戦略と実践 (2021/08/13)](https://www.m3tech.blog/entry/sudachi-es)
- [Elasticsearchのための新しい形態素解析器 「Sudachi」 (qiita) (2021/02/20)](https://qiita.com/sorami/items/99604ef105f13d2d472b)
- [ElasticsearchでSudachiとベクトル検索を組み合わせて使う方法 (2020/01/22)](https://www.ai-shift.co.jp/techblog/168)
- [Sudachi同義語辞書をElasticsearchで使う（暫定方法） (2021/01/26)](https://zenn.dev/sorami/articles/d19afb40fbb838)

## 手順

1. `$ docker exec -it sudachi-es01-1 sh`
2. `bin/elasticsearch-plugin install https://github.com/WorksApplications/elasticsearch-sudachi/releases/download/v3.1.0/elasticsearch-8.8.1-analysis-sudachi-3.1.0.zip`
    - > Plugin [analysis-sudachi] was built for Elasticsearch version 8.8.1 but version 8.10.4 is running
      - elasticsearch, kibanaのバージョンを8.8.1に下げる
3. `$ exit`
4. `$ docker restart sudachi-es01-1`
5. `$ docker exec -it sudachi-es01-1 sh`
6. `$ mkdir config/sudachi`
7. `$ exit`
8. `$ wget http://sudachi.s3-website-ap-northeast-1.amazonaws.com/sudachidict/sudachi-dictionary-20230927-full.zip`
9. `$ unzip sudachi-dictionary-20230927-full.zip`
10. `$ docker cp sudachi-dictionary-*/system_full.dic sudachi-es01-1:/usr/share/elasticsearch/config/sudachi/`
11. `$ docker restart sudachi-es01-1`


```json
{
  "number_of_shards": 1,
  "analysis": {
    "tokenizer": {
      "sudachi_tokenizer": {
        "type": "sudachi_tokenizer"
      }
    },
    "analyzer": {
      "sudachi_analyzer": {
        "filter": ["my_searchfilter" ],
        "tokenizer": "sudachi_tokenizer",
        "type": "custom"
      }
    },
    "filter":{
      "my_searchfilter": {
        "type": "sudachi_split",
        "mode": "search"
      }
    }
  }
}

```json
{
  "number_of_shards": 1,
  "analysis": {
    "tokenizer": {
      "sudachi_tokenizer": {
        "type": "sudachi_tokenizer",
        "split_mode": "C",
        "discard_punctuation": true,
        "resources_path": "/usr/share/elasticsearch/config/sudachii/"
      }
    },
    "analyzer": {
      "sudachi_analyzer": {
        "filter": [],
        "tokenizer": "sudachi_tokenizer",
        "type": "custom"
      }
    }
  }
}
```

```json
{
  "properties": {
    "原典資料コード": {
      "type": "long"
    },
    "大字・字・丁目区分コード": {
      "type": "long"
    },
    "大字町丁目コード": {
      "type": "long"
    },
    "大字町丁目名": {
      "type": "text",
      "analyzer": "sudachi_analyzer"
    },
    "市区町村コード": {
      "type": "long"
    },
    "市区町村名": {
      "type": "text",
      "analyzer": "sudachi_analyzer"
    },
    "経度": {
      "type": "double"
    },
    "緯度": {
      "type": "double"
    },
    "都道府県コード": {
      "type": "long"
    },
    "都道府県名": {
      "type": "text",
      "analyzer": "sudachi_analyzer"
    },
    "coordinates": {
      "type": "geo_point"
    }
  }
}
```
