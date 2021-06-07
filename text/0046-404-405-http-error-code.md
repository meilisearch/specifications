- Title: 404/405 Http Error Codes
- Start Date: 2021-06-07
- Specification PR: [#46](https://github.com/meilisearch/specifications/pulls/46)
- MeiliSearch Tracking-issues:


# 404/405 Http Error Codes

## 1. Feature Description and Interaction

### I. Summary

The purpose of this file is to specify the behavior of the API about the http code to return in case a resource does not exists or the http method used is not allowed on the resource.

This specification will be merged later into the more global API format specification that is currently being written.

### II. Motivation

A great API must be convenient to use. The development of a new endpoint, its improvement for developers must be done on a solid base to facilitate their work and focus on the intrinsic value of the endpoint. For users, it is also much easier and faster to use an API when endpoints share a common structure through conventions.

### III. Additional Materials
N/A

### IV. Explanation

### 404 Code

The API must return a `404` error code when the resource does not exist or when the url used does not allow to reach a resource.

E.g. Asking for an inexistent ressource.

**GET `/indexes/:index_uid/documents/2`**

In that case, the response body contain a `document_not_found` errorCode.

E.g.

```
{
    "message": "Document with id 2 not found",
    "errorCode": "document_not_found",
    "errorType": "invalid_request_error",
    "errorLink": "https://docs.meilisearch.com/errors#document_not_found"
}
```

E.g. Using a wrong url.

**GET `/indexess/documents/2`**

In that case, the response body should contain a `unrecognized_request_url` errorCode.

```
{
    "message": "Unrecognized request URL (GET: /indexess/documents/2). See https://docs.meilisearch.com/reference/api",
    "errorCode": "unrecognized_request_url",
    "errorType": "invalid_request_error",
    "errorLink": "https://docs.meilisearch.com/errors#unrecognized_request_url"
}
```
The message pattern is `Unrecognized request URL ({{ HTTP_VERB }}: {{REQUEST URL}}). See https://docs.meilisearch.com/reference/api`

### 405 Code

The API must return a `405` error code when the resource is reachable but the HTTP verb is not handled by the resource.

E.g. Using a http verb that is not allowed

**DELETE `/indexes/movies/search`**

In that case, the response body should contain a `method_not_allowed` errorCode.

```
{
    "message": "Method DELETE is not allowed. See https://docs.meilisearch.com/reference/api/search.html#search",
    "errorCode": "method_not_allowed",
    "errorType": "invalid_request_error",
    "errorLink": "https://docs.meilisearch.com/errors#method_not_allowed"
}
```

The message pattern is `Method {{HTTP_VERB}} is not allowed. See {{API REFERENCE LINK}}`

### V. Impact on Documentation

- Add `method_not_allowed` and `unrecognized_request_url` errors on the [errors documentation page](https://docs.meilisearch.com/errors/).

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities
N/A