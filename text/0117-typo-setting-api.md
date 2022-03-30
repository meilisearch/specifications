# Typo Tolerance Settings API

## 1. Summary

This specification describes the customizable options for the typo tolerance settings.

## 2. Motivation

Tolerance to typos is critical when building an enjoyable search experience. Meilisearch ranks typo-free matched documents above others based on the default settings that we believe are best for most users.

Meilisearch can adapt to different use-cases by providing customization options for the typo tolerance feature.

## 3. Functional Specification

### 3.1. `typo` API resource definition

| Field                                            | Type            | Required |
|--------------------------------------------------|-----------------|----------|
| [enabled](#311-enabled)                          | Boolean         | False    |
| [disabledAttributes](#312-disabledattributes)    | Array of String | False    |
| [disabledWords](#313-disabledwords)              | Array of String | False    |
| [minWordSizeForTypos](#314-minwordsizefortypos)  | Object          | False    |

#### 3.1.1. `enabled`

- Type: Boolean
- Required: False
- Default: `true`

Whether the typo tolerance feature is enabled.

##### 3.1.1.1. Impacts on the `typo` ranking rule

The presence of `typo` in the ranking rules setting does not influence the activation/deactivation of the typo tolerance feature.

If the `rankingRules` parameter of the index settings does not contain the `typo` rule, the results are not sorted according to the number of typos found.

The typo tolerance feature is still used to match documents.

### 3.1.2. `disabledAttributes`

- Type: Array of String
- Required: False
- Default: `[]`

`disabledAttributes` disable the typo tolerance feature on the specified attributes list.

Any attributes defined in `disabledAttributes` won't have their values matched by the typo tolerance.

#### 3.1.2.1. Example

Given `["title"]` as `disabledAttributes` and the following document

```json
{
    "id": 0,
    "title": "Hey World"
}
```

The engine won't try to match query term with typos with values of the `title` attribute at search time to match documents.

- Typing `Warld` won't bring the document as a candidate for the search results.

### 3.1.3. `disabledWords`

- Type: Array of String
- Required: False
- Default: `[]`

`disabledWords` disable the typo tolerance feature on a list of search query terms.

> This field is case insensitive since the document attributes values are lowercased and de-unicoded internally.

#### 3.1.3.1. Example

If `Javascript` is specified in `disabledWords`, the engine won't apply the typo tolerance on the query term `Javascript` or `javascript` if typed at search time to match documents.

### 3.1.4. `minWordSizeForTypos`

- Type: Object
- Required: False

#### 3.1.4.1. `minWordSizeForTypos` Object Definition

| Field                      | Type            | Required |
|----------------------------|-----------------|----------|
| [oneTypo](#3142-onetypo)   | Integer         | False    |
| [twoTypos](#3143-twotypos) | Integer         | False    |

#### 3.1.4.2. `oneTypo`

- Type: Integer
- Required: False
- Default: `5`

`oneTypo` customize the minimum size for a word to tolerate 1 typo.

> Given the default value `5`, the search engine starts to apply typo tolerance on a query term if its length is at least equal to 5 characters.

> See [2.1. Typos calculation section](#21-typos-calculation)

##### 3.1.4.2.1 Example

Given `5` as `oneTypo` and the following document

```json
{
    "id": 0,
    "title": "Hey World"
}
```

- Typing `World` with 1 typo, e.g. `Warld` will match `World`. It accepts 1 typo since `World` size is made of `5` chars.
- Typing `Hey` with 1 typo, e.g. `Hoy` won't match `Hey`. It accepts 0 typo since `Hey` size is made of `3` chars.

#### 3.1.4.3. `twoTypos`

- Type: Integer
- Required: False
- Default: `9`

`twoTypos` customize the minimum size for a word to tolerate 2 typos.

> Given the default value `9`, the search engine handles up to 2 typos on a query term if its length is at least equal to 9 characters.

> See [2.1. Typos calculation section](#21-typos-calculation)

##### 3.1.4.3.1. Example

Given `3` for `oneTypo` and `5` as `twoTypos` and the following document

```json
{
    "id": 0,
    "title": "Hey World"
}
```

- Typing `World` with 2 typos, e.g. `Warrld` will match `World`. It accepts up to 2 typos since `World` size is made of `5` chars.
- Typing `Hey` with 1 typo, e.g. `Hoy` will match `Hey`. It accepts only 1 typo since `Hey` size is made of `3` chars.

## 3.2. API Endpoints Definition

### 3.2.1. Global Settings API Endpoints Definition

`typo` is a sub-resource of `/indexes/:index_uid/settings`.

See [Settings API](0123-settings-api.md).

### 3.2.2. `indexes/:index_uid/settings/typo`

Manage the typo tolerance configuration for an index.

#### 3.2.2.1. `GET` - `indexes/:index_uid/settings/typo`

Allow fetching the current definition of the typo tolerance feature for an index.

`200` - Response body

```json
    {
        "enabled": true,
        "disabledAttributes": [],
        "disabledWords": [],
        "minWordSizeForTypos": {
            "oneTypo": 5,
            "twoTypos": 9
        }
    }
```

All properties must be returned when the resource is retrieved.

##### 3.2.2.1.2. Errors

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.2.2.2. `POST` - `indexes/:index_uid/settings/typo`

Allow customizing partially the settings of the typo tolerance feature for an index.

Request payload

```json
{
    "disabledAttributes": [
        "title",
        "description"
    ]
    "minWordSizeForTypos": 4
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

##### 3.2.2.2.1. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.2.2.2. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending a value with a different type than `Boolean` for the `enabled` field returns an [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending a value with a different type than `Array of String` for the `disabledAttributes` field returns an [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending a value with a different type than `Array of String` for the `disabledWords` field returns an [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending a value with a different type than `Integer` for `minWordSizeForTypos` object fields returns an [bad_request](0061-error-format-and-definitions.md#bad_request) error.

###### 3.2.2.2.2.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.2.2.2.1. Response Definition](#32221-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.2.3. Lazy Index Creation](#32223-lazy-index-creation).

- 🔴 Sending invalid integer values for the `minWordSizeForTypos` object fields returns an [invalid_typo_min_word_size_for_typos](0061-error-format-and-definitions.md#invalid_typo_min_word_size_for_typos) error.

##### 3.2.2.2.3. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.2.2.2.2.1. Async Errors](#322221-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.2.2.2.1. Response Definition](#32221-response-definition).

#### 3.2.2.3. `DELETE`- `indexes/:index_uid/settings/typo`

Allow resetting the typo tolerance feature to the default for an index.

##### 3.2.2.3.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.2.3.2. Errors

###### 3.2.2.3.2.1. Asynchronous Index Not Found Error

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related async `task` resource. See [3.2.2.3.1. Response Definition](#32231-response-definition).

### 3.2.3. General Errors

These errors apply to all endpoints described here.

#### 3.2.3.1. Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 2. Technical Details

### 2.1. Typos Calculation

The size of a query term triggers the typo tolerance feature.

By default, a query term made of `5` characters can tolerate `1` typo, while a query term made of `9` characters can tolerate `2` typos.

If a query term can be derived from a distance of one or two typos to be found in a document, it will be selected as a candidate. The number of typos found has an impact on the relevancy.

#### 2.1.1 Typo On The First Character

A query term having the first character considered as a typo will not count as `1` typo but as `2` typos.

It permits to drastically reduce the search space in this case, thus speed-up the search response.

#### 2.1.2. Clamping Typo On Concatened Query Terms

The concatenation of two query terms is equal to 1 typo.

e.g. If `Le tableau` is typed,  the concatened string `Letableau` made of `9` letters can only have `1` typo instead of `2` since `1` typo is already counted by the concatenation.

### 2.2. Typo Ranking Rule

The `typo` ranking rule favors candidates with the least typos. That is, if a document is found with two typos, it will be ranked as less important than a document found with 1 typo. 0 typo is the most favorable case.

## 3. Future Possibilities

- Expose `typo` resource as a search parameter to override index settings.
- Add the possibility to disable the typo tolerance feature on all numeric fields.
- Add different modes of result matching for the typo feature. e.g. `default`/`min`/`strict`
- Replace `POST` to `PATCH` verb to allow partial edit of the settings and embrace REST API convention.
- Introduce synchronous invalid_typo_{fieldName} error with a better error message than the one provided by serde.