# Compression Headers

## 1. Summary

Using the `Content-Encoding` header it's possible to send a compressed payload to Meilisearch. Using the `Accept-Encoding` header it's possible to receive a compressed response from Meilisearch.

## 2. Motivation

Compression HTTP headers can be used to improve transfer speed and to reduce bandwidth consumption by sending/receiving compressed, smaller payload.

## 3. Functional Specification

### 3.1. Supported Compression Algorithms

| name      |
|-----------|
| `gzip`    |
| `br`      |
| `deflate` |

### 3.2. Sending a compressed payload

Specify the algorithm used to compress the payload being sent to Meilisearch within the `Content-Encoding` header.

#### 3.2.1 Example with `gzip`

```cat ~/movies.json | gzip | curl -X POST 'http://localhost:7700/indexes/movies/documents' --data-binary @- -H 'Content-Type: application/json' -H 'Content-Encoding: gzip'```

### 3.3. Receiving a compressed response

Specify to Meilisearch the compression method to use when receiving a response within the `Accept-Encoding` header.

#### 3.3.1. Example with `gzip`

```curl -sH 'Accept-encoding: gzip' 'http://localhost:7700/indexes/movies/search' | gunzip -```

## 4. Technical Details
N/A

## 5. Future Possibilities

- Support `zstd` compression method.