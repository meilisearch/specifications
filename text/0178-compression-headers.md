# Compression Headers

## 1. Summary

Send and receive compressed payload, using the `Accept-Encoding` and `Content-Encoding` HTTP headers.

Meilisearch supports `gzip`, `deflate` and `brotli` compression methods.

## 2. Motivation

Compression HTTP headers can be used to improve transfer speed and to reduce bandwidth consumption by sending/receiving compressed, smaller payload.

## 3. Functional Specification

### 3.1. Supported Compression Algorithms

| name      |
|-----------|
| `gzip`    |
| `br` (refers to brotli) |
| `deflate` |

### 3.2. Sending a compressed payload

Specify the algorithm used to compress the payload being sent to Meilisearch within the `Content-Encoding` header.

See [RFC9110](https://httpwg.org/specs/rfc9110.html#field.content-encoding).

#### 3.2.1 Example with `gzip`

```cat ~/movies.json | gzip | curl -X POST 'http://localhost:7700/indexes/movies/documents' --data-binary @- -H 'Content-Type: application/json' -H 'Content-Encoding: gzip'```

### 3.3. Receiving a compressed response

Specify to Meilisearch the compression methods to use by order of preference when sending a response to a client within the `Accept-Encoding` header.

See [RFC9110](https://httpwg.org/specs/rfc9110.html#field.accept-encoding).

#### 3.3.1. Example with `gzip`

```curl -sH 'Accept-encoding: gzip' 'http://localhost:7700/indexes/movies/search' | gunzip -```

## 4. Technical Details
N/A

## 5. Future Possibilities

- Support `zstd` compression method.
