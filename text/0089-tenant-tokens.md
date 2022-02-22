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

- `Tenant tokens` are JWTs generated on the user side by using our SDKs or their own custom code. Thus not stored or retrievable on MeiliSearch side.
- `Tenant tokens` contain rules that ensure that a `Tenant token` holder (e.g an end-user) only has access documents matching rules chosen at the `tenant token` creation.
- `Tenant tokens` are signed from a MeiliSearch `API key` resource on the user's code.
- `Tenant tokens` cannot be less restrictive than the signing `API key` and can only be used for searching. A Tenant Token cannot search within more indexes than the API Key that signed that Tenant Token.
- `Tenant tokens` can have different rules for each index accessible by the signing API key. These filters rule per index are described in an `searchRules` json object.
- The only rule at the moment is the search parameter `filter`. Other rules may be added in the future.
- `Tenant tokens` are sent to Meilisearch via the `Authorization` header.
- When a search request is made with a Tenant Token, MeiliSearch decodes it to checks if the tenant token is authorized to make the request and then extract and inject the search rules for the search request being made.

### 1.2 Motivation

`Tenant tokens` are introduced to solve multi-tenant indexes use-case.

> Multi-Tenant Indexes Definition: It is an index that stores documents that may belong to different tenants. In our case, a tenant within an index can be a user or a company, etc.. In general, the data of one tenant should not be accessible by other tenants.

Users today need to set up workarounds to have multi-tenant indexes. Most of the time, they have to use server code to implement the access restriction logic before requesting MeiliSearch. It is difficult to maintain, to implement, and the performance is degraded because the frontend code does not communicate directly with MeiliSearch.

### 1.3 `Tenant Token` Explanations

#### 1.3.1 Example: Solving Multi-Tenancy with `Tenant tokens`

![](https://user-images.githubusercontent.com/3692335/151013496-d33ab507-f972-465d-b942-899fc2bd0a22.png)

`Mark` is a developer for a SaaS platform. He would like to ensure that every end-user can only access their documents at search time.

When an end-user registers, Mark's backend code generates a `Tenant token` for that end-user so they can only access their documents.

This tenant-token is signed with a MeiliSearch API Key so that MeiliSearch can ensure that the tenant-token has been generated from a known entity.

MeiliSearch checks if the Tenant Token is authorized to make the search request.

Then MeiliSearch extracts the Tenant Token's rules to apply for the search request.

## 2. Technical Details

### 2.1 `Tenant Token` details

A Tenant Token generated for MeiliSearch must respect several conditions.

#### 2.1.1 Header: Algorithm and token type

The Tenant Token MUST be signed with one of the following algorithms:

- `HS256`
- `HS384`
- `HS512`

e.g. With `HS256`

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

| Fields   | Required?  | Description | Comments |
| -------- |----------- | ----------- | -------- |
| `apiKeyPrefix` (Custom claim) | Required  | Must contain the first 8 characters of the signing `MeiliSearch API key` used to generate the JWT | |
| `exp` (Expiration Time claim) | Optional  | A JSON numeric value representing the number of seconds from 1970-01-01T00:00:00Z UTC until the specified UTC date/time. | If the signing API key expires, the Tenant Token also expires. Thus said, the `exp` can't be greater than the expiration date of the signing API key. |

##### 2.1.2.2 Business logic related

| Fields   | Required? | Description | Comments |
| -------- | --------- |------------ | -------- |
| `searchRules` | Required | This JSON object contains rules description to apply for search queries performed with the JWT depending the searched index. A Tenant Token cannot access more indexes at search time than those defined as accessible by the signing API key. | Let's say an index uses a field to separate documents belonging to one user from another one, but another index needs to separate belonging using a different field in its schema. Defining specific rules per accessible index avoids having to generate several tenant tokens for an end-user. |

##### 2.1.2.3 Payload example

Given a MeiliSearch API Key used to sign the JWT from the user code. Here is an example of a valid payload for a tenant token.

e.g `MeiliSearch API key: rkDxFUHd02193e120218f72cc51a9db62729fdb4003e271f960d1631b54f3426fa8b2595`

```jsonc
{
    "apiKeyPrefix": "rkDxFUHd", // The first 8 characters of the signing MeiliSearch API Key
    "exp": 1641835850, // An expiration date in seconds from 1970-01-01T00:00:00Z UTC
    "searchRules": { // The searchRules Json Object.
        "*": {
            "filter": "user_id = 1"
        }
    }
}
```

> In this example, `searchRules` allows to specify, that no matter which index is searched (among all those accessible by the signing API key that generated the tenant token), this filter will be applied on all search requests.

##### 2.1.2.4 `apiKeyPrefix` field

`apiKeyPrefix` permits to verify that the signing API key of the Token is known and valid within MeiliSearch. It must contain the first 8 characters of the MeiliSearch API key that generates and signs the Tenant Token.

The `apiKeyPrefix` can't be generated from a master-key.

##### 2.1.2.5 `exp` field

`exp` is used to specify the expiration date of the Tenant Token if needed. The format is a JSON numeric value representing the number of seconds from 1970-01-01T00:00:00Z UTC until the specified UTC date/time, ignoring leap seconds.

##### 2.1.2.6 `searchRules` JSON object

`searchRules` contains the rules to be enforced at search time for all or specific accessible indexes for the signing API Key.

Here are some valid examples in an attempt to cover all possible use cases.

---

> In this case, all indexes on which the signing API Key has permissions are searchable by the tenant token without any restrictions.

```json
{
    "searchRules": {
        "*": {}
    }
}
```

is equivalent to

```json
{
    "searchRules": {
        "*": null
    }
}
```

is equivalent to

```jsonc
{
    "searchRules": ["*"] //This notation does not allow the addition of specific rules. The search will just be accessible on all accessibles indexes from the signing API Key for the Tenant Token without specific rules.
}
```

---

> In this case, all searchable indexes from the signing API Key are searchable by the tenant token and MeiliSearch will apply the filter rule before applying the search parameters added by the end user.

```json
{
    "searchRules": {
        "*": {
            "filter": "user_id = 1"
        }
    }
}
```

---

> In this case, if the `medical_records` index is searchable from the signing API Key, the tenant token can only search in the `medical_records` index. No further rules impact search results on `medical_records`.

```json
{
    "searchRules": {
        "medical_records": {}
    }
}
```
is equivalent to

```json
{
    "searchRules": {
        "medical_records": null
    }
}
```

is equivalent to

```json
{
    "searchRules": ["medical_records"]
}
```

---

> In this case, if the `medical_records` index is searchable from the signing API Key, the tenant token can only search in the `medical_records` index, MeiliSearch will apply the filter rule before applying the search parameters added by the end user.

```json
{
    "searchRules": {
        "medical_records": {
            "filter": "user_id = 1"
        }
    }
}
```

---

> In this case, if the `medical_records` and `medical_appointments` indexes are searchable from the signing API Key, the tenant token can only search in those indexes. MeiliSearch will apply the filter rule before applying the search parameters added by the end user.

```json
{
    "searchRules": {
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

> In this case, all searchable indexes from the signing API Key are searchable and search requests will apply specific rules defined in the wildcard field: `*`. The `medical_appointments` index rules, defined in the field of the same name, overwrites the rules defined in the wildcard field `*` for this specific index.

```json
{
    "searchRules": {
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

Note: The `filter` field accepts array, string and the mixed syntax as described in the [filter and facet specification](0027-filter-and-facet-behavior.md).

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
    "apiKeyPrefix": meiliSearchApiKey.slice(0,8),
    "exp": 1641835850,
    "searchRules": {
        "*": {
            "filter": "user_id = 1"
        }
    }
}

base64Payload = base64Encode(payload)

hashedSignature = HS256(base64Header + '.' + base64Payload, meiliSearchApiKey)
signature = base64Encode(hashedSignature)


TenantToken = base64Header + '.' + base64Payload + '.' + signature
```

### 2.2 Revoking a Tenant Token

This is not possible at the moment to revoke a Tenant Token on the MeiliSearch side but may be added in the future.

For the moment the only way is to **delete the API key that signed it** using the `DELETE - /keys/:apiKey` endpoints of MeiliSearch.

🚨 **Doing this will revoke all tenant tokens signed by this API Key.**

Another much more drastic method is to modify the `master key` of the MeiliSearch instance.

🚨🚨 **Doing this will regenerate all the API Keys and thus revoke all the tenant tokens generated regardless of the signing API Key.**

## 3. Future Possibilities

- Handle more signing method for the Tenant Token.
- Handle more search parameters restrictions in `searchRules`.
- Find a solution to invalidate a specific Tenant Token.
- Find a solution to help error resolution / error making for a tenant token payload when a search request occurs. E.g. Help to locate that the error is coming from the `searchRules` `filter` field from a tenant token and not from the search request  `filter` parameter.