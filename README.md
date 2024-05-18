
# Policy QA System

This project is part of the 2nd round of evaluation for the Data Scientist position at Simpplr. The goal is to build a QA system to answer policy-related questions using RAG, making it easier for employees to quickly get answers to their queries.


## Table of Contents

 - [Introduction](#Introduction)
 - [Project Structure](#Project-Structure)
 - [Tech Stack](#Tech-Stack)
 - [Approaches](#Approaches)
 - [Evaluation Criteria](#Evaluation-Criteria)
 - [Setup and Installation](#Setup-and-Installation)
 - [API Endpoints](#API-Endpoints)
 - [Running Tests](#Running-Tests)


## Introduction
The Policy QA System is designed to answer questions about policies available in the provided pdf documents. The system utilizes an OpenAI model and Elasticsearch as vector store, providing responses and the sources from which the responses are generated.


## Project Structure
```bash
.
├── resources
│   ├── evaluation_rescords
│   │   ├── auto_merging_retreival.csv
│   │   ├── basic_rag.csv
│   │   └── elasticsearch_reranking.csv
│   ├── images
│   └── implementation_notebooks
│       ├── pdfs
│       │   ├── GPT - Expense Reimbursement Policy.pdf
│       │   ├── .
│       │   ├── .
│       │   └── GPT-parental leave policy.pdf
│       ├── auto_merging_retreival.ipynb
│       ├── basic_retrieval.ipynb
│       ├── elasticsearch_evaluation.ipynb
│       ├── knowledge_graph.ipynb
│       ├── functions.py
│       ├── evaluation_questions.txt
│       └── Simpplr DS Assignment.docx
├── main.py
├── test_functions.py
├── requirements.txt
├── Dockerfile
└── README.md

```


## Tech Stack

* **Frameworks and Libraries:** FastAPI, Uvicorn, Pandas, OpenAI API, TruLens, Llama Index

* **Vector Store:** Elasticsearch

* **Environment and Deployment:** Elasticsearch Docker Container, FastAPI Application Docker Container

* **Testing:** pytest


## Approaches
### Approach 1: Basic Retrieval-Based QA
* **Description**: Uses a simple `DocStore` to store and index documents and a basic retrieval mechanism to fetch top 3 relevant chunks.
* **Pros**: Simple, easy to implement.
* **Cons**: Limited accuracy, doesn't handle complex queries well.
* **Notebook**: basic_retrieval.ipynb

### Approach 2: Knowledge Graph-Based QA
* **Description**: Uses a `Graph Store` to store and index the documents and both keyword and embeddings to fetch relevant answers.
* **Pros**: Easy when query or documents have relation.
* **Cons**: Limited accuracy and high hallucinations.
* **Notebook**: knowledge_graph_rag.ipynb

### Approach 3: Auto Merging Retrieval-Based QA
* **Description**: Uses a `DocStore` and `HierarchicalNodeParser` to chunk, store and index the documents. It looks at a set of leaf nodes and recursively merges subsets of leaf nodes that reference a parent node beyond a given threshold allowing us to consolidate potentially disparate, smaller contexts into a larger context that might help synthesis.
* **Pros**: Better accuracy, context relevance and groundedness than basic retrieval.
* **Cons**: Higher latency than basic retrieval.
* **Notebook**: auto_merging_retrieval.ipynb

### Final Approach: Elasticsearch Retrieval with Reranking-Based QA
* **Description**: Uses `ElasticsearchStore` as vector store. Combines Elasticsearch with a reranking model (`BAAI/bge-reranker-base`) to improve the relevance of the retrieved chunks.
* **Pros**: Best accuracy, context relevance and groundedness among all approaches.
* **Cons**: High latency.
* **Script**: main.py
* **Evaluation Notebook**: elasticsearch_evaluation.ipynb


## Evaluation Criteria
The efficiency of the solution is evaluated based on:

* **Answer Relevance**: How relevant the answer is to the query.
* **Context Relevance**: How relevant the context of the answer is.
* **Groundedness**: The degree to which the answer is grounded in the source documents.
* **Latency**: The time taken to generate a response.
* **Total Tokens**: The number of tokens processed.
* **Total Cost**: The cost associated with processing the tokens.

*Note:* To evaluate any user query, you could use swagger ui by visiting:
  ```bash
  http://127.0.0.1:8000/docs
  ``` 


## Setup and Installation
Remove `\` and run in single line if throws error.

1) **Run Elasticsearch**
```bash
docker run -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  -e "xpack.security.http.ssl.enabled=false" \
  -e "xpack.license.self_generated.type=trial" \
  docker.elastic.co/elasticsearch/elasticsearch:8.9.0
```
2) **Run Policy_QA_System**
```bash

```


## API Endpoints

### Root Endpoint
Check if the FastAPI app is running.
* **URL**: `/`
* **Method**: `GET`
* **Response**:
```bash
{
  "message": "Policy QA System is running!",
  "endpoints": ["/generate/", "/evaluate/"],
  "methods": ["GET", "POST"],
  "parameters": ["user_query", "eval_question"],
  "access_swagger_ui": "http://127.0.0.1:8000/docs"
}

```

### Generate Response
Generate a response for a given user query.
* **URL**: `/generate/`
* **Method**: `POST`


| Parameter | Type     | Description                       |
| :-------- | :------- | :-------------------------------- |
| `user_query`      | `string` | The query provided by the user. |
* **Response**:
```bash
{
  "response": ...,
  "source_nodes": ...
}
```

### Evaluate Response
Evaluate the response for a given user query.

* **URL**: `/evaluate/`
* **Method**: `POST`


| Parameter | Type     | Description                       |
| :-------- | :------- | :-------------------------------- |
| `eval_question`      | `string` | The question to be evaluated. |
* **Response**:
```bash
[
  {
    "Answer Relevance": ...,
    "Context Relevance": ...,
    "Groundedness": ...,
    "latency": ...,
    "total_tokens": ...,
    "total_cost": ...,
    "input": ...,
    "output": ...
  }
]

```


## Running Tests

Run tests using pytest:
```bash
pytest
```
