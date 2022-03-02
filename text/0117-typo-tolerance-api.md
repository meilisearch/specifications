- Title: Typo Tolerance API
- Start Date: 2022-02-22

# Typo Tolerance API

## 1. Summary

This specification describes the customizable options for the typo tolerance settings.

## 2. Motivation

Tolerance to typos is critical when building an enjoyable search experience. Meilisearch ranks typo-free matched documents above others based on default settings that we believe are best for most users.

Meilisearch can adapt to different use-cases with specific requirements by providing customization options for the typo tolerance feature.

## 3. Functional Specification

### 3.1. `typoTolerance` API resource definition

| Field                | Type            | Required |
|----------------------|-----------------|----------|
| enabled              | Boolean         | True     |
| disableOnAttributes  | Array of String | True     |
| disableOnWords       | Array of String | True     |
| minWordSizeFor1Typo  | Integer         | True     |
| minWordSizeFor2Typos | Integer         | True     |


#### 3.1.1. `enabled`

- Type: Boolean
- Required: True
- Default: `true`

Whether the typo tolerance feature is enabled.

The presence of `typo` in the ranking rules setting does not influence the activation/deactivation of the typo tolerance feature.

If the `rankingRules` parameter of the index settings does not contain the `typo` rule, the results are not sorted according to the number of typos found. However, the typo tolerance feature is still used to match documents.

### 3.1.2. `disableOnAttributes`

- Type: Array of String
- Required: True
- Default: `[]`

`disableOnAttributes` disable the typo tolerance feature on the specified attributes set.

> Any attributes defined in `disableOnAttributes` won't have their values matched by the typo tolerance.


### 3.1.3. `disableOnWords`

- Type: Array of String
- Required: True
- Default: `[]`

`disableOnWords` disable the typo tolerance feature for a set of search query terms.

> If `Javascript` is specified in `disableOnWords`, the engine won't apply the typo tolerance on the query term `Javascript` when typed at search time.

### 3.1.4. `minWordSizeFor1Typo`

- Type: Integer
- Required: True
- Default: `5`

`minWordSizeFor1Typo` customize the minimum size for a word to tolerate 1 typo.

> Given the default value `5`, the search engine starts to apply typo tolerance on a query term if its length is at least equal to 5 characters.

### 3.1.5. `minWordSizeFor2Typos`

- Type: Integer
- Required: True
- Default: `9`

`minWordSizeFor2Typos` customize the minimum size for a word to tolerate 2 typos.

> Given the default value `9`, the search engine handles up to 2 typos on a query term if its length is at least equal to 9 characters.

## 3.2. API Endpoints Definition

### 3.2.1 `indexes/:index_uid/settings/typo-tolerance`

Manage the typo tolerance configuration for an index.

#### 3.2.1.1 `GET` - `indexes/:index_uid/settings/typo-tolerance`

Allow fetching the current definition of the typo tolerance feature for an index.

`200` - Response body

```json
    {
        "enabled": true,
        "disableOnAttributes": [],
        "disableOnWords": [],
        "minWordSizeFor1Typo": 5,
        "minWordSizeFor2Typos": 9
    }
```

##### 3.2.1.1.2 Errors

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.
- 🔴 If secured, accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 If secured, accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

#### 3.2.1.2 `POST` - `indexes/:index_uid/settings/typo-tolerance`

Allow customizing the default settings of the typo tolerance feature for an index.

Request payload

```json
{
    "enabled": true,
    "disableOnAttributes": [
        "title",
        "description"
    ],
    "disableOnWords": [
        "Tolkien"
    ],
    "minWordSizeFor1Typo": 4,
    "minWordSizeFor2Typos": 8
}
```

`202 Accepted` - Example Response

```json
{
    "uid": 42,
    "indexUid": "books",
    "status": "enqueued",
    "type": "settingsUpdate",
    "enqueuedAt": "2022-03-01T18:39:29.228155Z"
}
```

> Returns a 202 response. See [Tasks API](0060-tasks-api.md)

##### 3.2.1.2.1 Errors

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.
- 🔴 If secured, accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 If secured, Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

#### 3.2.1.3 `DELETE`- `indexes/:index_uid/settings/typo-tolerance`

Allow resetting the typo tolerance feature to the default for an index.

> Returns a 202 response. See [Tasks API](0060-tasks-api.md)

##### 3.2.1.3.1 Errors

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.
- 🔴 If secured, accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 If secured, Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

### 3.3.1 `indexes/:index_uid/settings`

> See [Settings API]

## 2. Technical Details
tbd

## 3. Future Possibilities

- Expose `typoTolerance` properties at search time.
- Add the possibility to disable typo-tolerance on all numeric fields.
- Add different modes of result matching for the typo-tolerance feature. e.g. `default`/`min`/`strict`
- Replace `POST` to `PATCH` verb to allow partial edit of the settings and embrace REST API convention.