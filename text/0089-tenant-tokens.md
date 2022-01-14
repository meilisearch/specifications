- Title: Tenant Tokens
- Start Date: 2021-10-15
- Specification PR: [#89](https://github.com/meilisearch/specifications/pull/89)
- Discovery Issue: [#51](https://github.com/meilisearch/product/issues/51)

# Tenant Token

## 1. Functional Specification

### 1.1 Summary

A `Tenant token` is generated on the user code to be used by an end-user. It allows users to have multi-tenant indexes and thus restricts access to documents depending on the end-user making the search request given enforced rules specified from the user business logic.

A Tenant Token is a JWT containing the information necessary for MeiliSearch to verify it and extract permission/rules to apply it to the end user's search.

#### 1.1.1 Summary Key Points

- `Tenant tokens` are JWTs generated on the user side. Thus not stored or retrievable on MeiliSearch side.
- `Tenant tokens` contain rules that ensure that a `Tenant token` holder (e.g an end-user) only has access documents matching rules chosen at the `tenant token` creation.
- `Tenant tokens` are signed from a MeiliSearch `API key` on the user's code.
- `Tenant tokens` cannot be less restrictive than the signing `API key` and can only be used for searching. A Tenant Token cannot search within more indexes than the API Key that signed that Tenant Token.
- `Tenant tokens` can have different rules for each index accessible by the signing API key. These filters rule per index are described in an `indexesPolicy` json object.
- The only rule at the moment is the search parameter `filter`. Other rules may be added in the future.
- When a request is made with the Tenant Token, MeiliSearch checks if this Tenant Token is authorized to make the request and in this case injects the rules at search time.

### 1.2 Motivation

`Tenant tokens` are introduced to solve multi-tenant use-cases.

Users today need to set up workarounds to have multi-tenant indexes. Most of the time, they have to use server code to implement the access restriction logic before requesting MeiliSearch. It is difficult to maintain, to implement, and the performance is degraded because the frontend code does not communicate directly with MeiliSearch.

### 1.3 `Tenant Token` Explanations

#### 1.3.1 Example: Solving Multi-Tenancy with `Tenant tokens`

![](https://user-images.githubusercontent.com/3692335/149373631-de6f3c5f-a514-4c8d-b018-ee09ccaeaf4d.png)

`Mark` is a developer for a SaaS platform. He would like to ensure that every end-user can only access their documents at search time.

When an end-user registers, the Mark's backend code generates a `Tenant token` for that end-user so they can only access their documents.

This tenant-token is signed with a MeiliSearch API Key so that MeiliSearch can ensure that the tenant-token has been generated from a known entity.

MeiliSearch check if the Tenant Token is authorized to make the search request.

Then MeiliSearch extract the Tenant Token's rules to apply for the search request.

## 2. Technical Details

### 2.1 `Tenant Token` details

A Tenant Token generated for MeiliSearch must respect several conditions.

#### 2.1.1 Header: Algorithm and token type

Tenant Token MUST be signed with the combination of `HMAC + SHA256`.

e.g.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

#### 2.1.2 Payload: Data

MeiliSearch needs information within the tenant token to check its validity and use it to authorize and perform end-user requests.

This information can be separated into two parts, on one hand, the information allows to check the validity of the Tenant Token, on the other hand, the business logic information allows to apply search parameters / rules for the end user's search request.

##### 2.1.2.1 Validity related

| Fields   | Description | Comments |
| -------- | -------- | -------- |
| `iss` (Issuer claim) | Must contain the first 8 characters of the signing `MeiliSearch API key` used to generate the JWT |  | |
| `exp` (Expiration Time claim) | A JSON numeric value representing the number of seconds from 1970-01-01T00:00:00Z UTC until the specified UTC date/time, ignoring leap seconds. | This value is optional. |

##### 2.1.2.2 Business logic related

| Fields   | Description | Comments |
| -------- | -------- | -------- |
| `indexesPolicy` | This JSON object contains rules description to apply for search queries performed with the JWT depending the searched index. | Let's say an index uses a field to separate documents belonging to one user from another one, but another index needs to separate belonging using a different field in its schema. Defining specific rules per accessible index avoids having to generate several tenant tokens for an end-user. |

##### 2.1.2.3 Payload example

Given a MeiliSearch API Key used to sign the JWT from the user code. Here is an example of a valid payload for a tenant token.

e.g `MeiliSearch API key: rkDxFUHd02193e120218f72cc51a9db62729fdb4003e271f960d1631b54f3426fa8b2595`

```json
{
    "iss": "rkDxFUHd", <- The first 8 characters of the signing API Key
    "exp": 1641835850, <- An expiration date in seconds from 1970-01-01T00:00:00Z UTC
    "indexesPolicy": { <- The indexesPolicy Json Object.
        "*": {
            "filter": "user_id = 1"
        }
    }
}
```

> In this example, `indexesPolicy` allows to specify, that no matter which index is searched (among all those accessible by the signing API key that generated the tenant token), this filter will be applied on all search requests.

##### 2.1.2.4 `iss` field

##### 2.1.2.5 `exp` field

##### 2.1.2.6 `indexesPolicy` object

`indexesPolicy` is a description of the possible rules for each index.

Here are some valid examples in an attempt to cover all possible use cases.

---

> In this case, all indexes searchable from the signing API Key will be searchable by the tenant token without specific rules.

```json
{
    "indexesPolicy": {}
}
```

is equivalent to

```json
{
    "indexesPolicy": {
        "*": {}
    }
}
```

---

> In this case, all indexes searchable from the signing API Key will be searchable by the tenant token and MeiliSearch will apply the filter definition before applying the search parameters added by the end user.

```json
{
    "indexesPolicy": {
        "*": {
            "filter": "user_id = 1"
        }
    }
}
```

---

> In this case, if the medical_records index is searchable from the signing API Key, the tenant token can only search in the medical_records index without applying specific rules before applying the search parameters added by the end user.

```json
{
    "indexesPolicy": {
        "medical_records": {}
    }
}
```

---

> In this case, if the medical_records index is searchable from the signing API Key, the tenant token can only search in the medical_records index and specific rules will be applied at search time before applying the search parameters added by the end user.

```json
{
    "indexesPolicy": {
        "medical_records": {
            "filter": "user_id = 1"
        }
    }
}
```

---

> In this case, if the medical_records and medical_appointments indexes are searchable from the signing API Key, the tenant token can only search in those indexes and will apply specific rules given the searched index before applying the search parameters added by the end user.

```json
{
    "indexesPolicy": {
        "medical_records": {
            "filter": "user_id = 1"
        },
        "medical_appointments": {
            "filter": "user_id = 1 AND accepted = true"
        }
    }
}
```

---

> In this case, all searchable indexes from the signing API Key will be searchable and the rules under `*` will be applied at search time. The medical_appointments index policy will replace the `*` rules specifically when this index is searched by those specified for that index.

```json
{
    "indexesPolicy": {
        "*": {
            "filter": "user_id = 1"
        },
        "medical_appointments": {
            "filter": "user_id = 1 AND accepted = true"
        }
    }
}
```

---

### 2.2 `Tenant Token` Javascript Code Sample

```javascript

meiliSearchApikey = 'rkDxFUHd02193e120218f72cc51a9db62729fdb4003e271f960d1631b54f3426fa8b2595';

header = {
  "alg": "HS256",
  "typ": "JWT"
}

base64Header = base64Encode(header)

payload = {
    "iss": meiliSearchApiKey.slice(0,8),
    "exp": 1641835850,
    "indexesPolicy": {
        "*": {
            "filter": "user_id = 1"
        }
    }
}

base64Payload = base64Encode(payload)

signature = HS256(base64Header + '.' + base64Payload, meiliSearchApiKey)

TenantToken = base64Header + '.' + base64Payload + '.' + signature
```

## 3. Future Possibilities

- Handle more signing method for the Tenant Token.
- Handle more search parameters restrictions in `indexesPolicy`.