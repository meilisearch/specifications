- Title: API Format
- Start Date: 2021-06-03
- Specification PR: [#45](https://github.com/meilisearch/specifications/pull/45)
- MeiliSearch Tracking-issues:

# API Format

## 1. Feature Description and Interaction

### I. Summary

This specification is intended to become a convention about a standardized format for the search engine API.

The specification will address in the following order:

[Naming conventions and consistency](#naming-conventions-and-consistency)
[Request parameters format](#request-parameters-format)
[Response body format](#response-body-format)
[Enumeration convention](#enumeration-convention)

### II. Motivation

A great API must be convenient to use. The development of a new endpoint, its improvement for developers must be done on a solid base to facilitate their work and focus on the intrinsic value of the endpoint. For users, it is also much easier and faster to use an API when endpoints share a common structure through conventions.

### III. Additional Materials
N/A

### IV. Explanation

### Naming conventions and consistency

> Quick acces
> - [Endpoints naming convention](#endpoints-naming-convention)
> - [Request parameters naming convention](#request-parameters-naming-convention)
> - [Response body naming convention](#response-body-naming-convention)

A 2010 [study](https://ieeexplore.ieee.org/document/5521745) confirms that developers find it easier to read the snake_case than the camelCase. Nevertheless, the increasing use of javascript seems to have introduced and accustomed developers to the use of camelCase.

`camelCase`: This method has been democratized by the Java language. It consists in marking the beginning of each word by putting the first letter in a lower case. For example: camelCase, currentUser, addAttributeToGroup, etc. **Apart from the expert debates on readability, its main flaw is that it is unusable in case-insensitive contexts**.

`snake_case`: This method has been used for a long time in C and more recently in Ruby. The words are separated by underscore `_` allowing a compiler or interpreter to understand it as a single symbol, but allowing the human reader to separate the words in an almost natural way. **Unlike camel case, there are very few contexts in which it is incompatible.**

`spinal-case`: A variation of snake_case, spinal case uses short dashes "-" to separate words. The advantages and disadvantages are largely identical to snake case, except that many programming languages cannot accept it as a symbol (variable name or function).

### 1. Endpoints naming convention

Here are the naming conventions for resource in endpoints of massively used APIs.

| API endpoints naming convention            | Twitter | Github | Stripe | Spotify | Algolia | Paypal | Shopify |
|--------------------------------------------|---------|--------|--------|---------|---------|--------|---------|
| camelCase                                  |         |        |        |         | x       |        |         |
| snake_case                                 | x       |        | x      |         |         |        |x        |
| spinal-case                                |         |  x     |        | x       |         | x      |         |

The current version of the MeiliSearch API manages resources naming in `spinal-case` notation. E.g. `/indexes/:index_uid/settings/ranking-rules`.

🟢 **Specification Proposal: Replace the spinal-case format by a snake_case format.**

E.g.

`/indexes/:index_uid/settings/ranking-rules`

becomes

`/indexes/:index_uid/settings/ranking_rules`

### 2. Request parameters naming convention

In general, the request parameter name convention is the same as the respondy body parameters convention.

### 3. Response body naming convention

Here are the naming conventions for parameters in the response body of massively used APIs.

| Response Body Parameters Naming convention | Twitter | Github | Stripe | Spotify | Algolia | Paypal | Shopify |
|--------------------------------------------|---------|--------|--------|---------|---------|--------|---------|
| camelCase                                  |         |        |        |         | x       |        |         |
| snake_case                                 | x       | x      | x      | x       |         |x       |x        |
| spinal-case                                |         |        |        |         |         |        |         |

The current version of the MeiliSearch API manages parameters in `camelCase` notation.

E.g. Response body from `/indexes/:index_uid/updates/{update_id}`
```
{
  "status": "processed",
  "updateId": 1,
  "type": {
    "name": "DocumentsAddition",
    "number": 4
  },
  "duration": 0.076980613,
  "enqueuedAt": "2019-12-07T21:16:09.623944Z",
  "processedAt": "2019-12-07T21:16:09.703509Z"
}
```

🟢 **Specification Proposal: Replace the camelCase format by a snake_case format.**

This is a format for which developers are more accustomed to when using an API. It also follows the proposal to have url ressources named as `snake_case` and allows to unify the whole API on the same naming rule.

E.g.

```
{
  "status": "processed",
  "updateId": 1,
  "type": {
    "name": "DocumentsAddition",
    "number": 4
  },
  "duration": 0.076980613,
  "enqueuedAt": "2019-12-07T21:16:09.623944Z",
  "processedAt": "2019-12-07T21:16:09.703509Z"
}
```

becomes

```
{
  "status": "processed",
  "update_id": 1,
  "type": {
    "name": "DocumentsAddition",
    "number": 4
  },
  "duration": 0.076980613,
  "enqueued_at": "2019-12-07T21:16:09.623944Z",
  "processed_at": "2019-12-07T21:16:09.703509Z"
}
```

---

### Request parameters format

> Quick acces
> - [Filters](#filters)
> - [Sorting](#sorting)
> - [Fields Selection](#fields-selection)

#### Filter

##### Filter parameter syntax

On some routes we would like to offer filtering options based on the attributes we deem useful.

We therefore propose to use a `filter` attribute that will be easily plugged in as we go along on all the routes. As we also use the `filter` field on the `/search` endpoint. We keep the same name to indicate a similar intent even though the syntax may change between the `filter` set of standard routes compared to `/search` for business purposes.

Here is the syntax we propose for filters on numeric and alphanumeric attributes.

- Filters can be separated by the `,` character.
- Filters can offer a set of operations like `=`, `>`, `>=`, `<`, `<=`.

E.g I want to filter the list of updates `/updates` by status.

`filter="status=processed"`

E.g. I want to filter on a nested field value like the `name` from the update `type` object.

`filter="type.name=DocumentAdditions"`

E.g. I want to combine those filters.

`filter="type.name=DocumentAdditions,status=processed"`

E.g. I want DocumentAdditions stricly greater than `1000` documents.

`filter="type.name=DocumentAdditions,type.number>=1000"`

> Do we need to return an error if the client try filter on an inexistent field? It may be interesting to give the list of filter capability for the resource in the given error.

#### Sorting

On some routes we would like to offer sorting according to the attributes we consider useful.

We therefore propose to use a `sort` attribute that will be easily plugged in as we go along on all the routes. As we also use the `sort` field on the `/search` endpoint. We keep the same name to indicate a similar intent and thus facilitate the use of the API as a whole.

Here is the syntax we propose for sorting on numeric and alphanumeric attributes.

- Sort criterion can be separated by the `,` character.
- Sort can offer `asc` and `desc` orders.

E.g. I want to sort the list of updates `/updates` by `duration` in descending order.

`sort="duration:desc"`

> Do we need to return an error if the client try to sort by an inexistent field? It may be interesting to give the list of sort capability for the resource in the given error.

#### Field Selection

On some routes we would like to offer the possibility to display specific `fields`.

We therefore propose to use a `fields` parameter that will be easily plugged in as we go along on all the routes.

Here is the syntax we propose for fields selection.

- Object fields can be separated by the `,` character.
- It is possible to eliminate an object field from the response with `-` operator.

E.g. I want to get the list of updates`/updates` only with `status` and `updateId` on the `update` object.

`fields="updateId,status"`

E.g. I want all fields of an update except `enqueudAt`

`fields="-enqueudAt"`

By default `fields` is equivalent to `*` in order to display all public fields of an object.

> Do we need to return an error if the client try to display/remove inexistent field? It may be interesting to give the list of field list for the resource in the given error.
---

### Response body format

> Quick acces
> - [Object identifier case](#object-identifier-case)
> - [Null value case](#null-value-case)
> - [Date and Time](#date-and-time-format)
> - [Body Response blocks](#body-response-blocks)
> - [Body Response blocks - data](#data)
> - [Body Response blocks - pagination](#pagination)

### Object identifier case

We would like to add a readable object identifier to easily contextualize API results in response.

All response should now contain an `object` field to indicate the type of the resource.

E.g. Getting a single `update` item.

```
{
    "object": "update",
    "status": "processed",
    "updateId": 0,
    "type": {
        "name": "DocumentsPartial",
        "number": 19546
    },
    "duration": 17.405,
    "enqueuedAt": "2021-07-20T20:17:41.026784Z",
    "processedAt": "2021-07-20T20:17:58.432491Z"
}
```

E.g. Getting multiple `update` items.

```
{
  "object": "update",
  "data": [
    {
      "status": "processed",
      "updateId": 0,
      "type": {
          "name": "DocumentsPartial",
          "number": 19546
      },
      "duration": 17.405,
      "enqueuedAt": "2021-07-20T20:17:41.026784Z",
      "processedAt": "2021-07-20T20:17:58.432491Z"
    }
  ]
}
```

> 💡 Maybe we could have a reserved `_object` instead of `object` not to be confused with the attributes that are added by users in the documents.

### Null and Empty value case
TBD

### Date and Time

We specify the date and time with the `ISO 8601` standard. Example of what we use as a value in an `update` object for the `processedAt` field.
`2021-07-20T20:17:58.432491Z`

### Body Response blocks

#### Multiple items response

All responses must be represented by a JSON object. That is, responses containing a list of items are now placed in an array called `data` or `items` (TBD). This allows us to add additional response parameters for paging e.g. with `limit` or `offset` or the number of elements returned and thus give a conventional key to easily find and parse query results.

It also allows us to add in the future global attributes contextualized by the response objects. For example in our case with the list of `updates`, we could add a `duration` field at the first level of the response that accumulates all the `duration` of the update items.

```
{
  "object": "update",
  "duration": //duration sum of attribute in data items. This is just an example.
  "data": [
    {
      "status": "processed",
      "updateId": 0,
      "type": {
          "name": "DocumentsPartial",
          "number": 19546
      },
      "duration": 17.405,
      "enqueuedAt": "2021-07-20T20:17:41.026784Z",
      "processedAt": "2021-07-20T20:17:58.432491Z"
    },
    ...
  ],
  "count": 10,
  "limit": 10,
  "offset": 0
}
```

#### Single item response

This is crystal clear from the previous point. The only variation is that the `object` field is lowered into the object.

```
{
  "object": "update",
  "status": "processed",
  "updateId": 0,
  "type": {
      "name": "DocumentsPartial",
      "number": 19546
  },
  "duration": 17.405,
  "enqueuedAt": "2021-07-20T20:17:41.026784Z",
  "processedAt": "2021-07-20T20:17:58.432491Z"
}
```

#### Pagination
`limit` `offset` TBD

### Status Enumeration Convention

Today we have several routes that do not use the same words to represent the same state.

A `dump` status

```
[
  "in_progress",
  "failed",
  "done"
]
```

An `update` status
```
[
  "enqueued",
  "processing",
  "processed"
  "failed"
]
```

We propose to have a common status enum definition to each object displaying an evolutive status.

```
[
  "enqueued",
  "processing",
  "done"
  "failed"
]
```

We can make this status evolve in the future but it is important that all APIs share the same naming convention for a similar state.

## 2. Technical Aspects
N/A

## 3. Future Possibilities
