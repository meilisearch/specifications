- Title: API Keys
- Start Date: 2021-10-15
- Specification PR: [#85](https://github.com/meilisearch/specifications/pull/85)
- Discovery Issue: [#51](https://github.com/meilisearch/product/issues/51)

# API Keys

## 1. Functional Specification

### 1.1 Summary

Granular management of API keys is added to MeiliSearch. It is possible to restrict the access of an API key to certain actions on specific indexes.

### 1.2 Motivation

To make MeiliSearch more reliable for teams, we extend the management and the possibilities of restrictions for the management of a MeiliSearch instance by introducing a concrete API reséource (`API Key`). Security is a critical need, often tricky to negotiate as the stakes are high for a company.

### 1.3 Glossary

| Term               | Definition |
|--------------------|------------|
| Master Key         | This is the master key that allows you to create other API keys. The master key is defined by the user when launching MeiliSearch, thus gives access to the `/keys` API endpoint. |
| API Key            | API keys are stored and managed from the endpoint `/keys` by the master key holder. These are the keys used by the technical teams to interact with MeiliSearch at the level of the client code. |

### 1.4 Personas

| Persona | Role |
|---------|------|
| Anna    | Anna has an `ssh` access to the MeiliSearch instance and manages it on a daily basis. |
| Mark    | Mark is a developer from the same company as Anna. He will implement the code to communicate with MeiliSearch to solve technical/product needs. |

### 1.5 `API Key` Explanations

#### 1.5.1 Summary Key Points

- `X-MEILI-API-KEY` header is replaced by the `Authorization` header. `API keys` must be specified with the bearer authorization method. See examples.
- `/keys` management is restricted to the master key.
- When a master key is set at MeiliSearch first-launch, we generate two pre-configured default `API Key` resources. A `Default Search API Key` restricted to the search action and a `Default Admin API Key` to handle all operations (except managing `/keys` resource) on MeiliSearch.
- If the master-key changes, all `API Keys` are re-generated.
- These default API Keys can be modified/deleted with the `/keys` endpoint but are not re-created if MeiliSearch has already generated them at some point.
- New endpoints are added to manage the `API Key` resource.
- `API keys` can have restrictions on which methods can be accessed via an `actions` list; they can also `expiresAt` a specific date time and be restricted to a specific set of `indexes`.
- There is no possibility to regenerate the value of the `key` field for an `API key` in this first iteration.
- New errors are added and the `missing_authorization_header` message is updated.
- The current state of the `API Keys` resource is propagated to snapshots and dumps.

#### 1.5.2 Master Key

The master key exists to secure a MeiliSearch instance. As soon as a master key is set via the  `MEILI_MASTER_KEY` environment variable or the `--master-key` CLI option , the endpoint `/keys` is accessible only for the master key holder. It can be seen as a super admin key; It must be shared only with people who have to manage the security of a MeiliSearch instance.

This master key is not an API Key, thus is not stored and fetchable from the `/keys` API endpoint. It must be seen as a runtime lock that activates the security of MeiliSearch as soon as an instance is launched with it.

At the first launch of MeiliSearch with a master key, MeiliSearch automatically generates two default API keys (See the `GET - /keys` example) to cover many of the most basic needs. It generates a `Default Search API Key` dedicated to the search that can be used on the client-side and a `Default Admin API Key` to manipulate a MeiliSearch instance from a backend side.

If the master key is removed at MeiliSearch launch, the previously generated API keys no longer secure the MeiliSearch instance.

If MeiliSearch is launched with the `production` value for the `MEILI_ENV` environment variable or the `--env` CLI option, a master key is mandatory to force the user to secure his instance. If the master key is omitted in that particular case, MeiliSearch launch is aborted and displays the `Error: In production mode, the environment variable MEILI_MASTER_KEY is mandatory` error in stdout.

> 🚨 A user coming from a version prior to v0.25.0 and having a master-key will have to update his code to use the newly generated default keys replacing the public and private, since we changed the previous `public` and `private` API Key generation, is it mandatory for that specific version upgrade. Now, a prefix of an API Key is generated uniquely and the final value of the `key` field is a hash of that randomized prefix with the master-key. See 2.1 API Key Generation part.

> 🚨 The master-key should never be exposed to the public or bad-intentioned persons for security measures as it may compromise a MeiliSearch instance.

> 🚨 If the value of the master key changes, all the previously generated `API Keys` changes, thus allows to invalidate the set of keys previously generated by regenerating to a different value for the `key` field. This is particularly useful in the case where the master key might have been leaked and the user need to re-generate the whole set of keys at once to re-secure the instance.

> Note that the master key does not appear on the `/keys` endpoints.

#### 1.5.3 Default API Keys

When the user accessing the machine launches MeiliSearch with a `master` key the first time, MeiliSearch will generate two API keys described below, as it did before with the `public` and `private` key.

If the user changes the value of the `master` key later, these two default keys are not created again but re-generated with a different `key` field. However, these two API keys can be changed using the `/keys` endpoints.

MeiliSearch must know that it has already generated these Default API Keys internally so if the user delete them, the engine should not create them again when MeiliSearch is launched again with a `master` key.

##### 1.5.3.1 Default Search API Key

The `Default Search API key` gives access to the same rights as the old `public` key.

Here is how the `Default Search API Key` is represented after its generation.

```json
{
    "description": "Default Search API Key (Use it to search from the frontend)",
    "key": "0a6e572506c52ab0bd6195921575d23092b7f0c284ab4ac86d12346c33057f99", //example
    "actions": [
        "search"
    ],
    "indexes": [
        "*"
    ],
    "expiresAt": null,
    "createdAt": "2021-08-11T10:00:00Z", //example
    "updatedAt": "2021-08-11T10:00:00Z"
}
```

##### 1.5.3.2 Default Admin API Key

The `Default Admin API key` gives access to the same rights as the old `private` key.

Here is how the `Default Admin API Key` is represented after its generation.

```json
{
    "description": "Default Admin API Key (Use it for all other operations. Caution! Do not use it on a public frontend)",
    "key": "380689dd379232519a54d15935750cc7625620a2ea2fc06907cb40ba5b421b6f", //example
    "actions": [
        "*"
    ],
    "indexes": [
        "*"
    ],
    "expiresAt": null,
    "createdAt": "2021-08-11T10:00:00Z", //example
    "updatedAt": "2021-08-11T10:00:00Z"
}
```

#### 1.5.3 Managing `API Key`

![](https://i.imgur.com/mAUFnNb.png)

`Anna` manages the MeiliSearch instance; she uses the master key she defined at startup to generate an `API Key` resource for `Mark` to use in his client code to communicate with the MeiliSearch instance.

`Anna` can define access rights to certain indexes to define an expiration date and also authorized `actions` for an `API Key` (See API Key Actions List Definition Part).

Only the master key allows managing the API keys.

#### 1.5.4 `API Key` object representation

| field       | type    | description                                      |
|-------------|---------|--------------------------------------------------|
| description | string  | A description for the key. `null` if empty. |
| key         | string  | The generated key. **Generated by MeiliSearch**. |
| actions     | array   | A list of actions permitted for the key. `["*"]` for all actions. See Actions list definition part. |
| indexes     | array   | A list of indexes permitted for the key. `["*"]` for all indexes. |
| expiresAt   | string  | Represent the expiration date and time as `ISO-8601` format. `null` equals to no expiration time. |
| createdAt   | string  | Represent the date and time as `ISO-8601` format when the API key has been created. **Generated by MeiliSearch** |
| updatedAt   | string  | Represent the date and time as `ISO-8601` format when the API key has been updated. **Default**: Value of `createdAt`. **Generated by MeiliSearch** |

#### 1.5.5 `POST`/ `PATCH` - `/keys` - API Key object payload definition

| field       | type    | required |description                     |
|-------------|---------|----------|--------------------------------|
| indexes     | array   | Required | `[*]` for all indexes. **Default**: `No Default` |
| description | string  | Optional | A description for the API key. **Default**: `null` |
| actions     | array   | Required | A list of actions permitted for the API key. `["*"]` for all actions. **See Actions list definition part**. `*` character can be used as a wildcard. e.g. `documents.*` to authorize access on all documents endpoints. **Default**: `No default` |
| expiresAt   | string  | Required | The expiration date and time as `ISO-8601` format. `null` equals to no expiration time. Sending only the date part e.g `2021-12-01` leads to having an `expiresAt` value set to `2021-12-01T00:00:00`. **Default**: `No Default` |


#### 1.5.6 Actions List Definition

> `:authorizedIndexes` can be any value extracted from the `indexes` field of an `API key` resource.

| name    | description |
|---------|-------------|
| search  | Provides access to `GET` and `POST` methods on `/indexes/:authorizedIndexes/search` routes. |
| documents.add | Provides access to `POST` and `PUT` on `/indexes/:authorizedIndexes/documents` routes. |
| documents.get | Provides access to `GET` methods on `/indexes/:authorizedIndexes/documents` and `/indexes/:authorizedIndexes/documents/:documentId` routes. |
| documents.delete | Provides access to `DELETE` method on `/indexes/:authorizedIndexes/documents/:documentId`, `indexes/:authorizedIndexes/documents/:documentId` and `POST` method on `/indexes/:authorizedIndexes/documents/delete-batch` routes. |
| indexes.add | Provides access to `POST` `/indexes`. **⚠️ A newly created `index` is added to the `indexes` array for the API key making the operation and all others keys having `[*]` as a value for `indexes`**. |
| indexes.get | Provides access to `GET` `/indexes` and `/indexes/:authorizedIndexes`. **⚠️Non-authorized `indexes` are omitted from the response on `/indexes`**. |
| indexes.update | Provides access to `PUT` `/indexes/:authorizedIndexes`. |
| indexes.delete | Provides access to `DELETE` `/indexes/:authorizedIndexes`. |
| tasks.get | Provides access to `GET` `/tasks`. **⚠️Non-authorized `indexes` are omitted from the response on `/tasks`**. Also add access to `GET` `/indexes/:authorizedIndexes/tasks` routes. |
| settings.get | Provides access to `GET` `/indexes/:authorizedIndexes/settings` and `/indexes/:authorizedIndexes/settings/*` routes. |
| settings.update | Provides access to `POST / DELETE` `/indexes/:authorizedIndexes/settings` and `/indexes/:authorizedIndexes/settings/*` routes. |
| stats.get | Provides access to `GET` `/stats/`. **⚠️Non-authorized `indexes` are omitted from the response on `/stats`**. Also add access to `GET` `/indexes/:authorizedIndexes/stats`. |
| dumps.create | Provides access to `POST` `/dumps` route. **As dumps are not scoped by indexes, a restriction on `indexes` does not affect this action.** |
| dumps.get | Provides access to `GET` `/dumps/:dumpUid` route. **As dumps are not scoped by indexes, a restriction on `indexes` does not affect this action.** |
| version | Provides access to `GET` `/version` route.

---

#### **As `Anna 👩`, I want to create an `API key` for `Mark 👨🏻` client-code, so that he can index some documents into MeiliSearch**

##### Request Definition

`POST` - `/keys`

##### Headers

```
"Authorization: Bearer :masterKey"
"Content-Type: application/json"
```

##### Body Payload

```json
{
    "description": "Indexing Products API key",
    "actions": [
        "documents.add"
    ],
    "indexes": ["products"],
    "expiresAt": "2021-11-13T00:00:00Z"
}
```

##### Response

`201 Created`

```json
{
    "description": "Indexing Products API key",
    "key": "d0552b41536279a0ad88bd595327b96f01176a60c2243e906c52ac02375f9bc4",
    "actions": [
        "documents.add"
    ],
    "indexes": ["products"],
    "expiresAt": "2021-11-13T00:00:00Z",
    "createdAt": "2021-11-12T10:00:00Z",
    "updatedAt": "2021-11-12T10:00:00Z"
}
```

##### Requirements

- `actions` is mandatory and should be an array of valid `actions`.
- `indexes` is mandatory and should be an array of string.
- `expiresAt` is mandatory and must be a valid `ISO 8601` datetime in the future or `null`.
- If set, `description` should be a string or `null`.

##### Errors

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending a different payload type than the Content-Type header returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid json format returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Omitting `actions` field from the payload returns a [missing_parameter](0061-error-format-and-definitions.md#missing_parameter) error.
- 🔴 Omitting `indexes` field from the payload returns a [missing_parameter](0061-error-format-and-definitions.md#missing_parameter) error.
- 🔴 Omitting `expiresAt` field from the payload returns a [missing_parameter](0061-error-format-and-definitions.md#missing_parameter) error.
- 🔴 Sending an invalid value for the `actions` field returns an [invalid_api_key_actions](0061-error-format-and-definitions.md#invalid_api_key_actions) error.
- 🔴 Sending an invalid value for the `indexes` field returns an [invalid_api_key_indexes](0061-error-format-and-definitions.md#invalid_api_key_indexes) error.
- 🔴 Sending an invalid value for the `expiresAt` field returns an [invalid_api_key_expires_at](0061-error-format-and-definitions.md#invalid_api_key_expires_at) error.
- 🔴 Sending an invalid value for the `description` field returns an [invalid_api_key_description](0061-error-format-and-definitions.md#invalid_api_key_description) error.

---

#### **As `Anna 👩`, I want to get details about an `API Key`**

##### Request Definition

`GET` - `/keys/:key`

##### Headers

```
"Authorization: Bearer :masterKey"
```

##### Response

`200 Success`

```json
{
    "description": "Indexing API key",
    "key": "d0552b41536279a0ad88bd595327b96f01176a60c2243e906c52ac02375f9bc4",
    "actions": [
        "documents.add"
    ],
    "indexes": [
        "products"
    ],
    "expiresAt": "2021-11-13T00:00:00Z",
    "createdAt": "2021-11-12T10:00:00Z",
    "updatedAt": "2021-11-12T10:00:00Z"
}
```

##### Errors

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Attempting to access an API key that does not exist returns an [api_key_not_found](0061-error-format-and-definitions.md#api_key_not_found) error.

---

#### **As `Anna 👩`, I want to update API key to change its restrictions**

##### Request Definition

`PATCH` - `/keys/:key`

##### Headers

```
"Authorization: Bearer :masterKey"
"Content-Type: application/json"
```

##### Body Payload

> PATCH method allows making partial changes to an existing resource. Thus the user is not obliged to send the complete API resource for each update.

```json
{
    "description": "Manage Products/Reviews Documents API key", //Update the description to specify the API Key purposes.
    "actions": [
        "documents.add",
        "documents.delete" //Has now access to documents deletion
    ],
    "indexes": [
        "products",
        "reviews"
    ], //Has now access to reviews
    "expiresAt": "2021-12-31T23:59:59Z" //Extended to the end of the year
}
```

##### Response

`200 Success`

```json
{
    "description": "Manage Products/Reviews Documents API key",
    "key": "d0552b41536279a0ad88bd595327b96f01176a60c2243e906c52ac02375f9bc4",
    "actions": [
        "documents.add",
        "documents.delete"
    ],
    "indexes": [
        "products",
        "reviews"
    ],
    "expiresAt": "2021-12-31T23:59:59Z",
    "createdAt": "2021-11-12T10:00:00Z",
    "updatedAt": "2021-10-12T15:00:00Z"
}
```

##### Errors

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Attempting to access an API key that does not exist returns a [api_key_not_found](0061-error-format-and-definitions.md#api_key_not_found) error.
- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending a different payload type than the Content-Type header returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid json format returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid value for the `actions` field returns an [invalid_api_key_actions](0061-error-format-and-definitions.md#invalid_api_key_actions) error.
- 🔴 Sending an invalid value for the `indexes` field returns an [invalid_api_key_indexes](0061-error-format-and-definitions.md#invalid_api_key_indexes) error.
- 🔴 Sending an invalid value for the `expiresAt` field returns an [invalid_api_key_expires_at](0061-error-format-and-definitions.md#invalid_api_key_expires_at) error.
- 🔴 Sending an invalid value for the `description` field returns an [invalid_api_key_description](0061-error-format-and-definitions.md#invalid_api_key_description) error.

---

#### **As `Anna 👩`, I want to list the API Keys**

##### Request Definition

`GET` - `/keys`

##### Headers

```
"Authorization: Bearer :masterKey"
```

##### Response

`200 Success`

```json
{
    "results": [
        {
            "description": "Manage Products/Reviews Documents API key",
            "key": "d0552b41536279a0ad88bd595327b96f01176a60c2243e906c52ac02375f9bc4",
            "actions": [
                "documents.add",
                "documents.delete"
            ],
            "indexes": [
                "products",
                "reviews"
            ],
            "expiresAt": "2021-12-31T23:59:59Z",
            "createdAt": "2021-10-12T00:00:00Z",
            "updatedAt": "2021-10-13T15:00:00Z"
        },
        {
            "description": "Default Search API Key (Use it to search from the frontend code)",
            "key": "0a6e572506c52ab0bd6195921575d23092b7f0c284ab4ac86d12346c33057f99",
            "actions": [
                "search"
            ],
            "indexes": [
                "*"
            ],
            "expiresAt": null,
            "createdAt": "2021-08-11T10:00:00Z",
            "updatedAt": "2021-08-11T10:00:00Z"
        },
        {
            "description": "Default Admin API Key (Use it for all other operations. Caution! Do not share it on the client side)",
            "key": "380689dd379232519a54d15935750cc7625620a2ea2fc06907cb40ba5b421b6f",
            "actions": [
                "*"
            ],
            "indexes": [
                "*"
            ],
            "expiresAt": null,
            "createdAt": "2021-08-11T10:00:00Z",
            "updatedAt": "2021-08-11T10:00:00Z"
        }
    ]
}
```

> Expired API keys cannot be found on the `/keys` endpoints. This can be handled in the future with an archiving system or something else. See Future Possibilities part.

> 👉 Note the two default generated API keys here. When a master key is set at MeiliSearch's launch, it generates two pre-configured `API Keys`. A Default Search API Key restricted to the search action on all indexes and a Default Admin API Key on all indexes to handle all operations (except managing API Keys).

##### List details

- `API Key` objects are returned in a `results` array.
- `API Keys` are ordered by `createdAt` in `desc` order. (Most recent first)
- Expired `API Keys` are omitted and no longer accessible. See Future Possibilities part.
- No pagination yet. See Future Possibilities part.

##### Errors

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

---

#### **As `Anna 👩`, I want to delete an `API Key`**

##### Request Definition

`DELETE` - `/keys/:key`

##### Headers

```
"Authorization: Bearer :masterKey"
```

##### Response

`204 No-Content`

##### Errors

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Attempting to access an API key that does not exist returns a `api_key_not_found`.

---

#### Using `API Key` on client-code

![](https://i.imgur.com/kRKcB43.png)

`Mark 👨🏻` receives the `API Key` transmitted by `Anna 👩` on a secured channel of their choice. He uses it to authenticate requests from the client code to MeiliSearch.

#### **As `Mark 👨🏻`, I am using an expired/deleted API Key**

#### Request example

`POST` - `/indexes/movies/search`

#### Headers

```
    "Authorization: Bearer :apiKey"
    "Content-Type: application/json"
```

#### Response

- 🔴 Accessing this route with an `API Key` that has expired or, been deleted returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

---

### **As `Mark 👨🏻`, I am using a valid API Key, but the permission set is not sufficient to access the requested API resource**

#### Request example

`POST` - `/indexes/movies/search`

#### Headers

```
    "Authorization: Bearer :apiKey"
    "Content-Type: application/json"
```

#### Response

- 🔴 Accessing this route with an `API Key` that don't have sufficient permissions to access it returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 2. Technical Aspects

### 2.1 API Key generation

A prefix of 8 characters is randomly generated per key and is stored within the instance.

The final key is then a SHA-2556 hash made of the prefix and the master-key concatenation.

`SHA-256(:randomizedPrefix, :master-key)` gives the final `key` field to use.

### 2.2 Synchronous write of `API Key` resources

Writing to `/keys` endpoints are synchronous in order to return errors directly to the user when he performs an operation on them.

### 2.3 Propagating `API Key` to a dump.

The generated API keys must also transit within a dump to facilitate the upgrade of a MeiliSearch instance.

> 🚨 As a reminder, dumps must be stored in secure areas not accessible to the public or unaccredited persons. In general, you should avoid moving them off the host machine or do so via a secure channel as a security measure.

If the dumps ever leak, the api keys cannot be spoofed from the dump inspection because it needs the master-key to have the full value of a valid API key. Only the randomized prefixes are propagated in the dumps.

### 2.4 Propagating `API Key` to snapshots.

The generated API keys must also transit within a snapshot to facilitate the recovery of a MeiliSearch instance.

> 🚨 As a reminder, snapshots must be stored in secure areas not accessible to the public or unaccredited persons. In general, you should avoid moving them off the host machine or do so via a secure channel as a security measure.

If the snapshot ever leak, the `API Keys` cannot be spoofed from the snapshot inspection because it needs the master-key to have the full value of a valid `API key`. Only the randomized prefixes are propagated in the snapshots.

### 2.5 API Keys storage size limit

The maximum size of the API key storage is `100GB`.

## 3. Future Possibilities

- Regenerate a specific `API Key`.
- Add a generated id field to paginate the list of API Key.
- Have an "archive" state where manually deleted API Keys can be restored for a certain amount of time.
- Add rate-limiting per API Key.
- A restriction on the maximum offset/limit.
- Add search parameters restrictions for an API Key.