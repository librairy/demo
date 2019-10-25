<p align="center"><img width=40% src="https://github.com/librairy/demo/blob/master/logo.png"></p>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![Docker-Engine](https://img.shields.io/badge/docker_engine-v19+-blue.svg)
![Docker-Compse](https://img.shields.io/badge/docker_compose-v3.0+-blue.svg)
[![Release Status](https://jitci.com/gh/librairy/demo/svg)](https://jitci.com/gh/librairy/demo)
[![GitHub Issues](https://img.shields.io/github/issues/librairy/demo.svg)](https://github.com/librairy/demo/issues)
[![License](https://img.shields.io/badge/license-Apache2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

A step-by-step guide to get the most of librAIry

# Requirements
[Docker-Engine](https://docs.docker.com/install/) and [Docker-Compose](https://docs.docker.com/compose/install/) are required.

# Installation
1. Clone this project
    ```
    git clone https://github.com/librairy/demo.git
    ```
2. and run it with docker
    ```
    docker-compose up
    ```
3. That's all!!  

# Services

### NLP Analysis [ [tool](http://librairy.linkeddata.es/nlp) ]

Efficient and easy way to analyze large amounts of multilingual texts through standard HTTP and TCP APIs.

Built on top of several NLP open-source tools it offers:
- Part-of-Speech Tagger (and filter)
- Stemming (Lemmas)
- Entity Recognizer
- Wikipedia Relations
- Wordnet Synsets

### Corpus Exploration [ [dashboard](http://localhost:8983/solr/banana) ]

Analyze document collections to discover the main hidden themes in their texts and create learning models that can be explored through HTTP Restful interfaces. These models can be used for large-scale document classification and information retrieval tasks.

`librAIry` natively supports indexing structured documents in CSV or [JSONL](http://jsonlines.org/), even GZ compressed. PDF documents or any other format supported by the [Solr load interface](https://lucene.apache.org/solr/guide/8_2/uploading-data-with-index-handlers.html#uploading-data-with-index-handlers) are also accepted.

Let's index a documents subset of [JRC-Acquis](https://ec.europa.eu/jrc/en/language-technologies/jrc-acquis) corpora. It is just a HTTP-POST request through the [/documents](http://localhost:8081) service (*default user:password is demo:2019*) with the following json (*Set your email account to be notified when all documents are indexed*):

```json
{
  "contactEmail": "<your@email.com>",
  "dataSink": {
    "format": "SOLR_CORE",
    "url": "http://solr:8983/solr/documents"
  },
  "dataSource": {
    "name":"jrc",
    "dataFields": {
      "id": "0",
      "name": "1",      
      "text": ["2"]
    },
    "filter":",",
    "format": "CSV",
    "offset": 1,
    "size": -1,
    "url": "https://raw.githubusercontent.com/librairy/demo/master/data/jrc-en.csv"
  }
}
```

Statistics and some graphs about the corpus are available in the [dashboard](http://localhost:8983/solr/banana).

Now, a probabilistic topic model wrapped by a HTTP-Restful API will be created from these documents, and published in DockerHub.

The following HTTP-POST request is required by the
[/topics](http://localhost:8081) service (*a DockerHub account is required*):
```json
{
  "name": "my-first-model",
  "description": "Collection of legislative texts (EN) from the European Union generated between years 1958 and 2006",
  "contactEmail": "<your@email.com>",
  "version": "<model-version>",
  "docker": {
    "email": "<dockerHub-account>",
    "password": "<dockerHub-password>",
    "repository": "<dockerHub-account>/<model-name>",
    "user": "<dockerHub-username>"
  },
  "parameters": {
    "topics": "20"
  },
  "dataSource": {
    "dataFields": {
      "id": "id",
      "text": ["txt_t"]
    },
    "filter":"source_s:jrc && lang_s:en",
    "format": "SOLR_CORE",
    "offset": 0,
    "size": -1,
    "url": "http://solr:8983/solr/documents"
  }
}
```

Once the notification email has been received, a Docker container with the service publishing the topics discovered in the corpus will be available in DockerHub, and it can be started using:
```ssh
docker run -it --rm -p 8585:7777 <docker-account>/<model-name>:<model-version>
```

### Thematic Annotations [ [service](http://localhost:8983/solr/banana) ]

Categorize texts with labels learned from them or from a different corpus. Our annotators are designed to generate annotations for each of the items inside big collections of textual documents, in a way that is computationally affordable and enables a semantic-aware exploration of the knowledge inside.

Let's annotate the corpus with the [JRC-model](http://librairy.linkeddata.es/jrc-en-model) created from [Eurovoc](http://eurovoc.europa.eu/) categories. The following HTTP-POST request to [http://localhost:8081/annotations](http://localhost:8081) should be made:

```json
{
  "contactEmail": "<your@email.com>",
  "dataSink": {
    "format": "SOLR_CORE",
    "url": "http://solr:8983/solr/documents"
  },
  "dataSource": {
    "name":"jrc",
    "dataFields": {
      "id": "id",
      "name": "name_s",
      "text": ["txt_t"]
    },
    "filter": "source_s:jrc && lang_s:en",
    "format": "SOLR_CORE",
    "offset": 0,
    "size": -1,
    "url": "http://solr:8983/solr/documents"
  },
  "modelEndpoint": "http://librairy.linkeddata.es/jrc-en-model"
}
```

Documents will be annotated from models in their respective languages. `librAIry` links documents across multi-lingual models.

### Cross-lingual Similarity [ [browser](http://localhost:8080) ]

Texts are linked from their semantic similarity through cross-lingual labels and hierarchies of multi-lingual concepts. Documents from multi-language corpora are efficiently browsed and related without the need for translation. They are described by hash codes that preserve the notion of topics and group similar documents.

Documents should be previously annotated to be semantically related. Using [Corpus Browser](http://localhost:8080), a navigation through the corpora by semantically similar documents can be performed, regardless of language, filtering those that do not meet an specific criteria.
