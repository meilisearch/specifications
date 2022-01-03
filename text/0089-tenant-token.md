- Title: Tenant Tokens
- Start Date: 2021-10-15
- Specification PR: [#89](https://github.com/meilisearch/specifications/pull/89)
- Discovery Issue: [#51](https://github.com/meilisearch/product/issues/51)

# Tenant Token

## 1. Functional Specification

### I. Summary

The SDKs can generate `Tenant tokens` inheriting from a MeiliSearch's `API key` generated to force filters during a search for an end-user. A `Tenant token` is generated on the user server-side code to be used by an end-user. It allows users to have multi-tenant indexes and thus restricts access to documents depending on the end-user making the search request.

### II. Motivation

`Tenant tokens` are introduced to solve multi-tenant use-cases. By providing the management of API Keys, we remove one of the last major deal-breakers that makes users not choose MeiliSearch as a solution despite all our advantages.

Users regularly request Multi-Tenant indexes. Users today need to set up workarounds to meet this need. Some of them implement reverse-proxy or managed authentication systems like Hasura or Kong to filter what can and cannot be read. Others decide to use server code as a facade to implement the access restriction logic before requesting MeiliSearch. It is difficult to maintain, less efficient, and requires necessary skills that not everyone has.

### III. Glossary

| Term               | Definition |
|--------------------|------------|
| Master Key         | This is the master key that allows the creation of other API keys. The master key is defined by the user when launching MeiliSearch. |
| API Key            | API keys are stored and managed from the endpoint `/keys` by the master key holder. These are the keys used by the technical teams to interact with MeiliSearch at the level of the frontend/backend code. |
| Tenant Token     | These tokens are not stored and managed by a MeiliSearch instance. They are generated for each end-user by the backend code from a MeiliSearch API Key. They are used by the end-users to only search the documents belonging to them. |
| Multi-Tenancy      | By multi-tenancy, we mean that an end-user only accesses data belonging to him within an index shared with other end-users. |

### IV. Personas

| Persona | Role |
|---------|------|
| Mark    | Mark is a developer for a SaaS company. He will implement the code to communicate with MeiliSearch to solve technical/product needs. |
| UserX   | UserX represents any end-user searching from the frontend interfaces provided by Anna and Mark's company. |

### V. `Tenant Token` Explanations

#### Summary Key Points

- `Tenant tokens` are generated from a MeiliSearch parent `API key` on the user's backend code. They are meant to resolve multi-tenancy by restricting access to data within an index according to the criteria chosen by the team managing the MeiliSearch instance.
- `Tenant tokens` cannot be less restrictive than their respective parent `API key` and can only be used for searching with a predefined forced filter field.
- `Tenant tokens` are not stored and thus not retrievable on MeiliSearch. This is why we highly advise setting an expiration time on them for security reasons.

#### Solving Multi-Tenancy with `Tenant tokens`

![](https://i.imgur.com/J4jVe1n.png)

Let's say that `Mark` is a developer for a SaaS platform. He would like to ensure that end-user can only access their documents at search time. **His database contains many users and he hopes to have many more in the future.**

When a user registers, the backend code generates a `Tenant token` specifically for that end-user so they can only access their documents.

The `filter` parameter is set to restrict the search for documents having an `user_id` attribute. This `filter` parameter is contained in the `Tenant token` payload and cannot be modified during the search by the end-user making the request. `filter` can be made of any valid filters. e.g. `user_id = 10 and category = Romantic`

This `Tenant token` is generated from a parent `API key` used to cipher the `Tenant token`. On the MeiliSearch side, the payload of the `tenant token` defines what data the user is allowed to retrieve, in order words, MeiliSearch grants authorization based on the permission defined in the Tenant token payload.

---

#### Generating a `Tenant token`

```javascript
const tenantTokenRestrictions = {
     "indexesPolicy": {
         "products": {
             "filter": "user_id = 1"
         },
         "reviews": {
             "filter": "user_id = 1 AND published = true"
         }
     },
    "expiresIn": 3600
}

export const generateTenantToken = () => {
  return (parentApiKey: string, restrictions: tenantTokenRestrictions): string => {
    //extract the 8 first characters of the parentApiKey
    const prefix = parentApiKey.substring(0,8);

    //serialize restrictions (indexesPolicies object and expiresIn)
    const queryParameters = serializeQueryParameters(restrictions);

    //create the secured part (parentApiKey + queryParameters)
    const securedKey = crypto
      .createHmac('sha256', parentApiKey)
      .update(queryParameters)
      .digest('hex');

    //return the generated `Tenant Token`
    return Buffer.from(prefix + securedKey + queryParameters).toString('base64');
  };
};
```

##### tenantTokenRestrictions

The format allows defining specific enforced search filters for accessible indexes (these indexes must be defined in the parent `Api Key` used to generate the `Tenant Token` and have the search action).

If the user does not want to define specific filters for each index accessible to the parent API Key, he can use the `*` index wildcard rule.

A policy per index allows overriding the `"*"` behavior.

A `Tenant token` also accepts a number of seconds in the `expiresIn` field until it expires. This field should be mandatory and explicitly set to `null` if no expiration time is needed.


```javascript
const tenantTokenRestrictions = {
     "indexesPolicy": {
         "*": { //all search on indexes different than reviews will have the enforced filter `user_id`
             "filter": "user_id = 1"
         },
         "reviews": {
             "filter": "user_id = 1 AND published = true"
         }
     },
    "expiresIn": null //No expiration time ⚠️ Is not recommended for security and quality of life reasons because the only way to revoke it is to delete the parent key
}
```

#### Validity

`Tenant tokens` expire or are revoked when the parent `API Key` is deleted or expires.

## 3. Future Possibilities

- Handle more search parameters restrictions.
- Extends `Tenant Token` to more than `search` action.