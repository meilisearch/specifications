# Synonyms Setting API

## 1. Summary

This specification describes the `synonyms` index setting API endpoints.

## 2. Motivation
N/A

## 3. Functional Specification


### 3.1. Explanations

If multiple words have an equivalent meaning in a dataset, specifying a list of synonyms will make search results more relevant.

In general, a search on a word will return the same results as a search on any of its synonyms. There is one exception to this rule. See [4.2. Multi-word Phrases](#42-multi-word-phrases) section.

All synonyms are lowercased and de-unicoded during the indexing process. See [4.1. Normalization](#41-normalization) section.

#### 3.1.1. Usage Examples

Meilisearch supports two types of synonym declarations.

##### 3.1.1.1. One-way Association

One-way association permits to declare one word to be synonymous with another, but not the other way around.

***Request payload `PUT`- `/indexes/proucts/settings/synonyms`***
```json
{
    "phone": [
        "iphone"
    ]
}
```

In this case, a search for `phone` will return documents containing `iphone` as if they contained the word `phone`.

However, in the case of a search for `iphone`, documents containing `phone` will be ranked lower in the results due to the `typo` ranking rule.

##### 3.1.1.2. Mutual Association

By associating one or more synonyms with each other, they will be considered the same in both directions.

***Request payload `PUT`- `/indexes/proucts/settings/synonyms`***
```json
{
    "shoe": [
        "boot",
        "slipper",
        "sneakers"
    ],
    "boot": [
        "shoe",
        "slipper",
        "sneakers"
    ],
    "slipper": [
        "shoe",
        "boot",
        "sneakers"
    ],
    "sneakers": [
        "shoe",
        "boot",
        "slipper"
    ]
}
```

When a search is done with one of these words, all synonyms will be considered as the same word and will appear in the search results.

### 3.2. Global Settings API Endpoints Definition

`synonyms` is a sub-resource of `/indexes/:index_uid/settings`.

See [Settings API](0123-settings-api.md).

### 3.3. API Endpoints Definition

Manipulate the `synonyms` setting of a Meilisearch index.

#### 3.3.1. `GET` - `indexes/:index_uid/settings/synonyms`

Fetch the `synonyms` setting of a Meilisearch index.

##### 3.3.1.1. Response Definition

- Type: Object
- Default: `{}`

##### 3.3.1.2. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.3.2. `PUT` - `indexes/:index_uid/settings/synonyms`

Modify the `synonyms` setting of a Meilisearch index.

##### 3.3.2.1. Request Payload Definition

- Type: Object / `null`

Setting `null` is equivalent to using the [3.3.3. `DELETE` - `indexes/:index_uid/settings/synonyms`](#333-delete---indexesindexuidsettingssynonyms) API endpoint.

##### 3.3.2.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.2.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a request payload value type different of `Object`,  or `null` returns an [invalid_settings_synonyms](0061-error-format-and-definitions.md#invalid_settings_synonyms) error.

###### 3.3.2.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.3.2.2. Response Definition](#3222-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.4. Lazy Index Creation](#3224-lazy-index-creation).

##### 3.3.2.4. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.3.2.3.1. Async Errors](#33231-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.3.2.2. Response Definition](#3322-response-definition).

#### 3.3.3. `DELETE` - `indexes/:index_uid/settings/synonyms`

Reset the `synonyms` setting of a Meilisearch index to the default value `{}`.

##### 3.3.3.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.3.3. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

###### 3.3.3.3.1. Asynchronous Index Not Found Error

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related async `task` resource. See [3.3.3.1. Response Definition](#3331-response-definition).

#### 3.3.4. General Errors

These errors apply to all endpoints described here.

##### 3.3.4.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details

### 4.1. Normalization

All synonyms are lowercased and de-unicoded during the indexing process.

### 4.1.1. Example

Consider a situation where `Résumé` and `CV` are set as synonyms.

***Request payload `GET`- `/indexes/jobs/settings/synonyms`***
```json
{
  "Résumé": [
    "CV"
  ],
  "CV": [
    "Résumé"
  ]
}
```

A search for `cv` would return any documents containing `cv` or `CV`, in addition to any that contain `Résumé`, `resumé`, `resume`, etc. unaffected by case or accent marks.

### 4.2. Multi-word Phrases

Multi-word phrases are treated differently than associations between individual words.

When a multi-word phrase is considered the synonym of another word or phrase, the exact search query will always take precedence over its synonym(s).

#### 4.2.1. Example

Suppose `San Francisco` and `SF` as synonyms with a mutual association


***Request payload `GET`- `/indexes/jobs/settings/synonyms`***
```json
{
    "San Francisco": [
        "SF"
    ],
    "SF": [
        "San Francisco"
    ]
}
```

In this case, a search for `SF`, results containing `San Francisco` will also be returned. However, they will be considered less relevant than those containing `SF`. The reverse is also true.

### 4.3. Three Words Limitation

Multi-word synonyms are limited to a maximum of three words.

For example, although `League of Legends` and `LOL` can be synonymous, it will not work for `The Lord of the Rings` and `LOTR`.

## 5. Future Possibilities

- Automatically declare mutual association