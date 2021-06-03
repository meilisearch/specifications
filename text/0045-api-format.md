- Title: API Format
- Start Date: 2021-06-03
- Specification PR: [#45](https://github.com/meilisearch/specifications/pull/45)
- MeiliSearch Tracking-issues:

# API Format

## 1. Feature Description and Interaction

### I. Summary

This specification is intended to become a convention about a standardized format for the different API endpoints.

The specification will address in the following order:

[Naming conventions](#naming-conventions)
[Request parameters format](#request-parameters-format)
[Response body format](#response-body-format)

### II. Motivation

A great API must be convenient to use. The development of a new endpoint, its improvement for developers must be done on a solid base to facilitate their work and focus on the intrinsic value of the endpoint. For users, it is also much easier and faster to use an API when endpoints share a common structure through conventions.

### III. Additional Materials
N/A

### IV. Explanation

### Naming conventions

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

E.g. Response body from `/indexes/:index_uid/updates/:updateId`
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

#### Filters
TBD

#### Sorting
TBD

---

### Response body format

> Quick acces
> - [Null value case](#null-value-case)
> - [Timestamp format](#timestamp-format)
> - [Body Response blocks](#body-response-blocks)
> - [Body Response blocks - data](#data)
> - [Body Response blocks - pagination](#pagination)

### Null value case
TBD

### Timestamp format
TBD

### Body Response blocks

#### Data

#### Pagination


### V. Impact on Documentation
### VI. Impact on SDKs

## 2. Technical Aspects
N/A

## 3. Future Possibilities
