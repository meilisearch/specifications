- Title: Reset Stop-words and Synonyms settings with null value
- Start Date: 2021-06-08
- Specification PR: [#47](https://github.com/meilisearch/specifications/pull/47)
- MeiliSearch Tracking-issues: [transplant/#217](https://github.com/meilisearch/transplant/issues/217)

# Reset Stop-words and Synonyms settings with a null value.

## 1. Functional Specification

### I. Summary

All Sub Settings `POST` endpoint accepts a null value in the http request body to reset the engine configuration to the default values. This is not the case for [Stop-words](https://docs.meilisearch.com/reference/api/stop_words.html#update-stop-words) and [Synonyms](https://docs.meilisearch.com/reference/api/synonyms.html#update-synonyms) settings endpoints.

### II. Motivation

This specification is written to make the above mentioned routes consistent with the others.

### III. Additional Materials
N/A

### IV. Explanation

#### Example

E.g. Reset Stop-Words setting with a null value on [POST Stop-Words Endpoint](https://docs.meilisearch.com/reference/api/stop_words.html#update-stop-words)

Given this stop-words configuration

```
[
    "of",
    "the"
]
```

As a User I send

```
null
```

To Expect

```
[]
```

#### [POST Stop-Words Endpoint](https://docs.meilisearch.com/reference/api/stop_words.html#update-stop-words)

Sending a `null` value in the request body like the given request body above should reset the `Stop-Words` setting to the engine default values.

The default value for `Stop-Words` is an empty array.

#### [POST Synonyms Endpoint](https://docs.meilisearch.com/reference/api/synonyms.html#update-synonyms)

Sending a `null` value in the request body like the given request body above should reset the `Synonyms` setting to the engine default values.

The default value for `Synonyms` is an empty object.

### V. Impact on Documentation
N/A

### VI. Impact on SDKs

The SDKs must allow the `null` value for every setting.

## 2. Technical Aspects

## 3. Future Possibilities
