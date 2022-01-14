- Title: Tenant Tokens
- Start Date: 2021-10-15
- Specification PR: [#89](https://github.com/meilisearch/specifications/pull/89)
- Discovery Issue: [#51](https://github.com/meilisearch/product/issues/51)

# Tenant Token

## 1. Functional Specification

### 1.1 Summary

A `Tenant token` is generated on the user server-side code to be used by an end-user. It allows users to have multi-tenant indexes and thus restricts access to documents depending on the end-user making the search request.

A Tenant Token is a JWT containing the information necessary for MeiliSearch to verify it and extract permission/rules to apply it to the end user's search.

#### 1.1.1 Summary Key Points

- `Tenant tokens` are meant to solve multi-tenancy in an elegant way.
- `Tenant tokens` contains rules that ensure that a `Tenant token` holder (e.g an end-user), only access to documents matching rules chosen at the `tenant token` creation.
- `Tenant tokens` are JWTs.
- `Tenant tokens` are signed from a MeiliSearch `API key` on the user's backend code.
- `Tenant tokens` cannot be less restrictive than the signing `API key` and can only be used for searching. It  mean that a Tenant Token cannot search within more indexes than the API Key that signed that token.
- `Tenant tokens` can have different filters for each index accessible by the signing API key. These filters rule per index are described in an `indexesPolicy` json object
- `Tenant tokens` are not stored and thus not retrievable on MeiliSearch.

### 1.2 Motivation

`Tenant tokens` are introduced to solve multi-tenant use-cases.

Users today need to set up workarounds to have multi-tenant indexes. Most of the time, they have to use server code as a frontend to implement the access restriction logic before requesting MeiliSearch. It is difficult to maintain, to implement, and the performance is degraded because the frontend code does not communicate directly with MeiliSearch.

### 1.3 `Tenant Token` Explanations

#### 1.3.1 Example: Solving Multi-Tenancy with `Tenant tokens`

![](https://user-images.githubusercontent.com/3692335/149373631-de6f3c5f-a514-4c8d-b018-ee09ccaeaf4d.png)

`Mark` is a developer for a SaaS platform. He would like to ensure that every end-user can only access their documents at search time.

When an end-user registers, the Mark's backend code generates a `Tenant token` for that end-user so they can only access their documents.

This tenant-token is signed with a MeiliSearch API Key so that MeiliSearch can ensure that the token is valid when search requests have to be authorized.

The `filter` parameter is set to restrict the search for documents having an `user_id` attribute. This `filter` parameter is part of the Tenant Token payload and cannot be modified.

`filter` can be made of any valid filters. e.g. `user_id = 10 and category = Romantic`

On the MeiliSearch side, the payload of the `tenant token` defines what data the user is allowed to retrieve, in order words, MeiliSearch grants authorization based on the permission defined in the Tenant token payload and force the `filter` field on top of any other filters that might have been set on the front-end;

## 2. Technical Details

### 2.1 `Tenant Token` details

A Tenant Token generated for MeiliSearch must respect several conditions. A Tenant Token is a JWT.

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

MeiliSearch needs information within the token to check its validity and use it to perform end-user requests.

This information can be separated into two parts, on one hand, the information allows to check the validity of the Tenant Token, on the other hand, the business logic information allows to have a token with pre-defined search parameters / rules for the end user's search.

##### 2.1.2.1 Validity related

| Fields   | Description | Comments |
| -------- | -------- | -------- |
| `iss` (Issuer claim) | Must contain the first 8 characters of the signing `MeiliSearch API key` used to generate the JWT |  | |
| `exp` (Expiration Time claim) | A JSON numeric value representing the number of seconds from 1970-01-01T00:00:00Z UTC until the specified UTC date/time, ignoring leap seconds. | This value is optional. |

##### 2.1.2.2 Business logic related

| Fields   | Description | Comments |
| -------- | -------- | -------- |
| `indexesPolicy` | This field contains descriptions of the rules applied for search queries performed with the JWT for all and specific indexes accessible by the signing API key used to generate the JWT. | Let's say an index uses a different field to separate documents belonging to one user from another one, but another index that needs to be accessible uses a different field in its schema. Being able to define specific rules per accessible index avoids having to generate several tenant tokens for an end-user.|

##### 2.1.2.3 Payload example

Given a MeiliSearch API Key used to sign the JWT from the user backend code. Here is a valid payload data for a JWT.

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

### 2.3 `Tenant Token` Javascript Code Sample

```javascript

meiliSearchApikey = 'rkDxFUHd02193e120218f72cc51a9db62729fdb4003e271f960d1631b54f3426fa8b2595';

header = {
  "alg": "HS256",
  "typ": "JWT"
}

base64Header = base64Encode(header)

payload = {
    "iss": meiliSearchApiKey.substr(8),
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

- Handle more signing method on MeiliSearch side.
- Handle more search parameters restictions in `indexesPolicy`.