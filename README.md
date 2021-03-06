<p align="center"><img src="https://github.com/elastic/app-search-python/blob/master/logo-app-search.png?raw=true" alt="Elastic App Search Logo"></p>

<p align="center"><a href="https://circleci.com/gh/elastic/app-search-python"><img src="https://circleci.com/gh/elastic/app-search-python.svg" alt="CircleCI build"></a>

> A first-party Python client for building excellent, relevant search experiences with [Elastic App Search](https://www.elastic.co/products/app-search).

## Contents

- [Getting started](#getting-started-)
- [Dependencies](#dependencies)
- [Versioning](#versioning)
- [Usage](#usage)
- [Running tests](#running-tests)
- [FAQ](#faq-)
- [Contribute](#contribute-)
- [License](#license-)

---

## Getting started 🐣

To install the client, use pip:

```python
python -m pip install elastic-app-search
```

You can also download the project source and run::

```python
python setup.py install
```

## Dependencies

- Python 2.7 / Python 3.3
- [Requests](https://github.com/requests/requests)
- [PyJWT](https://github.com/jpadilla/pyjwt)

## Versioning

This client is versioned and released alongside App Search.

To guarantee compatibility, use the most recent version of this library within the major version of the corresponding App Search implementation.

For example, for App Search `7.3`, use `7.3` of this library or above, but not `8.0`.

If you are using the [SaaS version available on swiftype.com](https://app.swiftype.com/as) of App Search, you should use the version 7.5.x of the client.

## Usage

### Instantiating a client

Using this client assumes that you have already an instance of [Elastic App Search](https://www.elastic.co/products/app-search) up and running.

The client can be instantiated using the `base_endpoint`, `api_key` and `use_https` parameters:

```python
>>> from elastic_app_search import Client
>>> client = Client(
    base_endpoint='localhost:3002/api/as/v1',
    api_key='private-mu75psc5egt9ppzuycnc2mc3',
    use_https=False
)
```

Notes:

The `[api_key]` authenticates requests to the API.
You can use any key type with the client, however each has a different scope.
For more information on keys, check out the [documentation](https://swiftype.com/documentation/app-search/api/credentials).

The `base_endpoint` must exclude the protocol and include the `api/as/v1` prefix. This can typically be found in the Credentials tab within the App Search Dashboard.

Set `use_https` to `True` or `False` depending how your server is configured. Often times it will be `False` when running in development on `localhost` and `True` for production environments.

The following is example of a configuration for Elastic Cloud:

```python
>>> from elastic_app_search import Client
>>> client = Client(
    base_endpoint='77bf13bc2e9948729af339a446b06ddcc.app-search.us-east-1.aws.found.io/api/as/v1',
    api_key='private-mu75psc5egt9ppzuycnc2mc3',
    use_https=True
)
```

#### Swiftype.com App Search users:

When using the [SaaS version available on swiftype.com](https://app.swiftype.com/as) of App Search, you can configure the client using your `host_identifier` instead of the `base_endpoint` parameter.
The `host_identifier` can be found within the [Credentials](https://app.swiftype.com/as#/credentials) menu.

```python
>>> from elastic_app_search import Client
>>> host_identifier = 'host-c5s2mj'
>>> api_key = 'private-mu75psc5egt9ppzuycnc2mc3'
>>> client = Client(host_identifier, api_key)
```


### Indexing: Creating or Updating a Single Document

```python
>>> engine_name = 'favorite-videos'
>>> document = {
      'id': 'INscMGmhmX4',
      'url': 'https://www.youtube.com/watch?v=INscMGmhmX4',
      'title': 'The Original Grumpy Cat',
      'body': 'A wonderful video of a magnificent cat.'
    }
>>> client.index_document(engine_name, document)
{'id': 'INscMGmhmX4'}
```

### Indexing: Creating or Updating Multiple Documents

```python
>>> engine_name = 'favorite-videos'
>>> documents = [
    {
      'id': 'INscMGmhmX4',
      'url': 'https://www.youtube.com/watch?v=INscMGmhmX4',
      'title': 'The Original Grumpy Cat',
      'body': 'A wonderful video of a magnificent cat.'
    },
    {
      'id': 'JNDFojsd02',
      'url': 'https://www.youtube.com/watch?v=dQw4w9WgXcQ',
      'title': 'Another Grumpy Cat',
      'body': 'A great video of another cool cat.'
    }
]

>>> client.index_documents(engine_name, documents)
[{'id': 'INscMGmhmX4', 'errors': []}, {'id': 'JNDFojsd02', 'errors': []}]
```

### Indexing: Updating documents (Partial Updates)

```python
>>> engine_name = 'favorite-videos'
>>> documents = [
    {
      'id': 'INscMGmhmX4',
      'title': 'Updated title'
    }
]

>>> client.update_documents(engine_name, documents)
```

### Get Documents

```python
>>> engine_name = 'favorite-videos'
>>> client.get_documents(engine_name, ['INscMGmhmX4'])
[{'id': 'INscMGmhmX4','url': 'https://www.youtube.com/watch?v=INscMGmhmX4','title': 'The Original Grumpy Cat','body': 'A wonderful video of a magnificent cat.'}]
```

### List Documents

```python
>>> engine_name = 'favorite-videos'
>>> client.list_documents(engine_name, current=1, size=20)
{
    'meta': {
        'page': {
        'current': 1,
        'total_pages': 1,
        'total_results': 2,
        'size': 20
        }
    },
    'results': [{'id': 'INscMGmhmX4','url': 'https://www.youtube.com/watch?v=INscMGmhmX4','title': 'The Original Grumpy Cat','body': 'A wonderful video of a magnificent cat.'}]
}
```

### Destroy Documents

```python
>>> engine_name = 'favorite-videos'
>>> client.destroy_documents(engine_name, ['INscMGmhmX4'])
[{'id': 'INscMGmhmX4','result': True}]
```

### Get Schema

```python
>>> engine_name = 'favorite-videos'
>>> client.get_schema(engine_name)
{'name':'text', 'square_km': 'number', 'square_mi': 'text'}
```

### Create/Update Schema

```python
>>> engine_name = 'favorite-videos'
>>> client.update_schema(engine_name, {'square_km': 'text'})
{'square_km': 'text'}
>>> client.update_schema(engine_name, {'square_mi': 'text'})
{'square_km': 'text', 'square_mi': 'text'}
>>> client.update_schema(engine_name, {'square_km': 'number'})
{'square_km': 'number', 'square_mi': 'text'}
```

### List Engines

```python
>>> client.list_engines(current=1, size=20)
{
    'meta': {
        'page': {
        'current': 1,
        'total_pages': 1,
        'total_results': 2,
        'size': 20
        }
    },
    'results': [{'name': 'favorite-videos'}, {'name': 'another-engine'}]
}
```

### Get an Engine

```python
>>> client.get_engine('favorite-videos')
{'name': 'favorite-videos'}
```

### Create an Engine

```python
>>> client.create_engine('favorite-videos', 'en')
{'name': 'favorite-videos', 'type': 'default', 'language': 'en'}
```

### Destroy an Engine

```python
>>> client.destroy_engine('favorite-videos')
{'deleted': True}
```

### List all synonym sets in an engine

#### With default pagination (a page size of 20)

```python
>>> client.list_synonym_sets('us-national-parks')
{
  'meta': {
    'page': {
      'current': 1,
      'total_pages': 1,
      'total_results': 3,
      'size': 20
    }
  },
  'results': [
    {
      'id': 'syn-5b11ac66c9f9292013220ad3',
      'synonyms': [
        'park',
        'trail'
      ]
    },
    {
      'id': 'syn-5b11ac72c9f9296b35220ac9',
      'synonyms': [
        'protected',
        'heritage'
      ]
    },
    {
      'id': 'syn-5b11ac66c9f9292013220ad3',
      'synonyms': [
        'hectares',
        'acres'
      ]
    }
  ]
}
```

#### With custom pagination

```python
>>> client.list_synonym_sets('us-national-parks', size=1, current=1)
{
  'meta': {
    'page': {
      'current': 1,
      'total_pages': 3,
      'total_results': 3,
      'size': 1
    }
  },
  'results': [
    {
      'id': 'syn-5b11ac66c9f9292013220ad3',
      'synonyms': [
        'park',
        'trail'
      ]
    }
  ]
}
```

### Get a single synonym set

```python
>>> client.get_synonym_set('us-national-parks', 'syn-5b11ac66c9f9292013220ad3')
{
  'id': 'syn-5b11ac66c9f9292013220ad3',
  'synonyms': [
    'park',
    'trail'
  ]
}
```

### Create a synonym set

```python
>>> client.create_synonym_set('us-national-parks', ['park', 'trail'])
{
  'id': 'syn-5b11ac72c9f9296b35220ac9',
  'synonyms': [
    'park',
    'trail'
  ]
}
```

### Update a synonym set

```python
>>> client.update_synonym_set('us-national-parks', 'syn-5b11ac72c9f9296b35220ac9', ['park', 'trail', 'ground'])
{
  'id': 'syn-5b11ac72c9f9296b35220ac9',
  'synonyms': [
    'park',
    'trail',
    'ground'
  ]
}
```

### Destroy a synonym set

```python
>>> client.destroy_synonym_set('us-national-parks', 'syn-5b11ac66c9f9292013220ad3')
{
  'deleted': True
}
```

### Searching

```python
>>> client.search('favorite-videos', 'grumpy cat', {})
{'meta': {'page': {'current': 1, 'total_pages': 1, 'total_results': 2, 'size': 10}, ...}, 'results': [...]}
```

### Multi-Search

```python
>>> client.multi_search('favorite-videos', [{
  'query': 'cat',
  'options': { 'search_fields': { 'title': {} }}
},{
  'query': 'dog',
  'options': { 'search_fields': { 'body': {} }}
}])
[{'meta': {...}, 'results': [...]}, {'meta': {...}, 'results': [...]}]
```

### Query Suggestion

```python
>>> client.query_suggestion('favorite-videos', 'cat', {
  'size': 10,
  'types': {
    'documents': {
      'fields': ['title']
    }
  }
})
{'results': {'documents': [{'suggestion': 'cat'}]}, 'meta': {'request_id': '390be384ad5888353e1b32adcfaaf1c9'}}
```

### Clickthrough Tracking

```python
>>> client.click(engine_name, {'query': 'cat', 'document_id': 'INscMGmhmX4'})
```

### Create a Signed Search Key

Creating a search key that will only search over the body field.

```python
>>> api_key = 'private-mu75psc5egt9ppzuycnc2mc3'
>>> api_key_name = 'my-api-token'
>>> signed_search_key = Client.create_signed_search_key(api_key, api_key_name, {'search_fields': { 'body': {}}})
>>> client = Client(
    base_endpoint='localhost:3002/api/as/v1',
    api_key=signed_search_key,
    use_https=False
)
```

### Create a Meta Engine

```python
>>> client.create_meta_engine(
    engine_name=engine_name,
    source_engines=[
        'source-engine-1',
        'source-engine-2'
    ]
)
{'source_engines': ['source-engine-1', 'source-engine-2'], 'type': 'meta', 'name': 'my-meta-engine'}
```

### Add a Source Engine to a Meta Engine

```python
>>> client.add_meta_engine_sources('my-meta-engine', ['source-engine-3'])
{'source_engines': ['source-engine-1', 'source-engine-2', 'source-engine-3'], 'type': 'meta', 'name': 'my-meta-engine'}
```

### Remove a Source Engine from a Meta Engine

```python
>>> client.delete_meta_engine_sources('my-meta-engine', ['source-engine-3'])
{'source_engines': ['source-engine-1', 'source-engine-2'], 'type': 'meta', 'name': 'my-meta-engine'}
```

### Get search settings

```python
>>> client.get_search_settings(engine_name='us-national-parks')
{
  "search_fields": {
    "name": {
      "weight": 1
    },
    "description": {
      "weight": 1
    }
  },
  "result_fields": {
    "name": {
      "raw": {}
    },
    "description": {
      "raw": {}
    }
  },
  "boosts": {}
}
```

### Update search settings

```python
>>> client.update_search_settings(
  engine_name='us-national-parks',
  search_settings={
    "search_fields": {
      "name": {
        "weight": 2
      },
      "description": {
        "weight": 1
      }
    },
    "result_fields": {
      "name": {
        "raw": {}
      },
      "description": {
        "raw": {}
      }
    },
    "boosts": {}
  }
)
{
  "search_fields": {
    "name": {
      "weight": 2
    },
    "description": {
      "weight": 1
    }
  },
  "result_fields": {
    "name": {
      "raw": {}
    },
    "description": {
      "raw": {}
    }
  },
  "boosts": {}
}
```

### Reset search settings

```python
>>> client.reset_search_settings(engine_name='us-national-parks')
{
  "search_fields": {
    "name": {
      "weight": 1
    },
    "description": {
      "weight": 1
    }
  },
  "boosts": {}
}
```

## Running tests

```python
python setup.py test
```

## FAQ 🔮

### Where do I report issues with the client?

If something is not working as expected, please open an [issue](https://github.com/elastic/app-search-python/issues/new).

### Where can I learn more about App Search?

Your best bet is to read the [documentation](https://swiftype.com/documentation/app-search).

### Where else can I go to get help?

You can checkout the [Elastic App Search community discuss forums](https://discuss.elastic.co/c/app-search).

## Contribute 🚀

We welcome contributors to the project. Before you begin, a couple notes:

- Prior to opening a pull request, please create an issue to [discuss the scope of your proposal](https://github.com/elastic/app-search-python/issues).
- Please write simple code and concise documentation, when appropriate.

## License 📗

[Apache 2.0](https://github.com/elastic/app-search-python/blob/master/LICENSE.txt) © [Elastic](https://github.com/elastic)

Thank you to all the [contributors](https://github.com/elastic/app-search-python/graphs/contributors)!
