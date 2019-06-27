---
title:  "Autocompleter with Elasticsearch"
# categories: tech
tags: Elasticsearch
---

# 상황
다음과 같이 autocomplete-v1라는 인덱스에 "고소한 참기름"이라는 text를 인덱싱했다고 가정해보자.
```
PUT autocomplete-v1
PUT autocomplete-v1/_doc/1
{
  "name":"고소한 참기름"
}
```

"고"와 같이 글자의 일부를 입력하여 이 document를 찾을 수 있을까?
```
GET autocomplete-v1/_search
{
  "query":{
    "match":{
      "name":"고"
    }
  }
}
```

결과는 찾을 수 없다. 이유는 "고소한 참기름"이라는 text는 Default tokenizer에 의해 "고소한"이라는 토큰과 "참기름"이라는 토큰으로 구분되어 인덱싱 될 뿐, 각 글자에 대해 인덱싱 되지는 않기 때문이다.
```
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

참고로 \_analze method를 사용하면 Default tokenizer로 어떻게 토크나이징이 되는지 알 수 있다. 
```
GET autocomplete-v1/_analyze
{
  "text":"고소한 참기름"
}
```

Default tokenizer로는 "고소한 참기름"이 "고소한" 이라는 토큰과 "참기름"이라는 토큰으로 구분됨을 확인하였다.
```
{
  "tokens": [
    {
      "token": "고소한",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<HANGUL>",
      "position": 0
    },
    {
      "token": "참기름",
      "start_offset": 4,
      "end_offset": 7,
      "type": "<HANGUL>",
      "position": 1
    }
  ]
}
```

# Edge NGram Tokenizer
ngram은 글자를 n글자로 나누는 방식이다.  (https://en.wikipedia.org/wiki/N-gram)
그 중 edge_ngram은 앞에서부터 나누기 때문에 자동완성에 효과적이다.
```
GET autocomplete-v1/_analyze
{
  "tokenizer": "edge_ngram", 
  "text":"고소한"
}
```

그러면 아래와 같이 한 글자, 또는 두 글자로 분해된 것을 확인할 수 있다. 따라서 이 Edge NGram Tokenizer를 자동완성을 위해 도입해보자.
```
{
  "tokens": [
    {
      "token": "고",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "고소",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 1
    }
  ]
}
```

# 방법
주의할 사항은 이미 인덱싱된 인덱스를 remapping하는 것이 불가능하므로, 새 인덱스에 reindexing해야한다. (인덱스를 유지하고 싶으면 alias 등의 방법을 써야한다.)

autocomplete-v2라는 인덱스를 새로 만들자. 이때 edge ngram tokenizer를 포함하고 있는 analyzer를 설정하여 ("autocomplete"로 명명) \_settings에 추가해야한다.min_gram은 1, max_gram은 3으로 세팅하여 한 글자에서 세 글자까지 분해하여 저장하도록 해보자.
```
PUT autocomplete-v2
{
  "analysis": {
    "analyzer": {
      "autocomplete": {
        "tokenizer": "autocomplete_tokenizer"
      }
    },
    "tokenizer": {
      "autocomplete_tokenizer": {
        "type": "edge_ngram",
        "min_gram": 1,
        "max_gram": 3,
        "token_chars": [
          "letter",
          "digit"
        ]
      }
    }
  }
}
```

이제 "name"이라는 필드에 우리가 만든 "autocomplete" analyzer를 추가하여 매핑을 해주자.
```
PUT autocomplete-v2/_mapping/_doc
{
  "properties": {
    "name": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      },
      "analyzer": "autocomplete"
    }
  }
}
```

이제 "autocomplete" analyzer가 "name" 필드에 잘 매핑이 되었으므로, reindexing할 때 우리가 만든 토크나이저를 거친 결과물이 인덱싱될 것이다.

```
POST _reindex
{
  "source": {"index": "autocomplete-v1"},
  "dest": {"index": "autocomplete-v2"}
}
```

Reindexing이 완료되었으면 "고"라는 글자를 입력하여 우리의 "고소한 참기름"이 잘 검색되는지 확인해보자.
```
GET autocomplete-v2/_search
{
  "query":{
    "match": {
      "name": "고"
    }
  }
}
```

"고소한 참기름" 검색이 잘 된다. 이제 글자 단위로 검색해도 결과가 검색되므로 자동완성에 활용할 수 있다.
```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "autocomplete-v2",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "name": "고소한 참기름"
        }
      }
    ]
  }
}
```

# Reference 
- Edge NGram Tokenizer
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html
