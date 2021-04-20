- Title: ASC / DESC Criterion
- Start Date: 2021-04-14
- Specification PR: [#30](https://github.com/meilisearch/specifications/pull/30)
- MeiliSearch Tracking-Issues: [#161](https://github.com/meilisearch/milli/issues/161)

## 1. Feature Description and Interaction

### I. Summary

Ranking rules are built-in rules that ensure relevancy in search results. Ranking rules are applied in a default order which can be changed in the settings. You can add or remove rules and change their order of importance.

MeiliSearch allows you to create custom rules within the default rules. Custom rules are dedicated to sorting in ascending or descending order on an attribute.

### II. Motivation

We want to provide our users with an always improved usage experience. Relevance is essential in a search engine since it is what allows the engine to fulfill its objective. Delivering results that match user demands by allowing modification of the relevance is critical.

The new search engine called Milli no longer processes this criterion of relevance the same way as the current MeilliSearch. This specification aims to make Milli identical both in API usage and in expected search results as the current ISO version (v0.20).

### III. Additional Materials

#### Algolia

Algolia offers 8 classification rules to achieve relevance.

- Number of typos
- Geolocation
- Number of words in the query matching in the result
- Filters
- Distance between words
- Best matching attribute in the record
- Number of words matching exactly (without typo)
- Custom ranking

Note that Algolia doesn't recommend changing the order of the default criteria because of the fact that it works for the vast majority of their use cases. Like MeiliSearch (v0.20), documents, where the attribute specified in the custom ranking is missing, are pushed to the bottom of the search results.

### IV.Explanation

#### Current behavior of v0.20

In the case of certain documents that do not contain the attribute configured on a custom ASC/DESC ranking rules, it will return those documents at the bottom of the search result. Documents are not excluded and can be returned in the search result.

> In the case of several custom ASC / DESC rules configured within the rankings rules, the sorting behavior is difficult to explain and can appear undefined when the documents do not necessarily contain the attributes which are set on this rule.

This criterion can only be used with an attribute containing numbers. 

#### Current behavior of Milli

If the attribute on which the ASC / DESC criterion is configured does not exist in all the documents of the index concerned by the search, Milli will discard them and never return them. This behavior is not necessarily what a end user would expect when searching.

As v0.20, Milli can’t handle ASC / DESC criterion on string. Only numbers allow its use.

#### Decisions

1. We thought about allowing the use of the ASC / DESC criterion in the ranking rules settings as long as it would also be declared in the attributesForFaceting.

✅ We have decided that Milli will act exactly as MeiliSearch: no need to declare the ranking rule field in `attributeForFaceting` since the rankings rules and the attributes for faceting do not strictly meet the same needs. Moreover, this could confuse the users configuring the search engine. The criterion configuration will remain on the `rankingRrules` field of the [global settings endpoint](https://docs.meilisearch.com/reference/api/settings.html#get-settings) and in the 
[specific ranking rules setting endpoint](https://docs.meilisearch.com/reference/api/ranking_rules.html).

2. We wondered if Milli's current behavior, which is to discard documents from search results that do not have the attribute configured as ascending and descending criteria might make sense for the purpose of a user performing a search.

✅ We have decided that Milli will act exactly as MeiliSearch concerning search results. The ASC / DESC criterion will no longer discard documents which do not have the attribute.

### V. Impact on documentation

- It should inform users about that having custom ASC / DESC criteria on attributes that are not always defined for each document may return results that are not necessarily relevant in the end user's eye.
- To be able to use this criterion on a date format, it must indicate that the timestamp is to be preferred since the type string is not supported.

### VI. Impact on SDKs
N/A

## 2. Technical Aspects

To apply the ranking rule, the search engine needs to create a database. This database is the same as we create to apply facets and filters on an attribute.
However, the users will not pass the attribute into `attributeForFaceting` when setting a ranking rule. It means the search engine must create this database for the related attribute.

ex:
```json
{
  "rankingRules": ["words", "typo", "asc(price)", "proximity"],
  "attributesForFaceting": ["genre"]
}
```json

means the search engine has to create facet databases for `genre` and `price`.

⚠️ Following this example, it also means the search engine would be technically able to apply filters and facet distribution on `price`, however, we should prevent this. To avoid confusion, the search engine should prevent the users to execute a filter or get facet distribution on the ranking attributes. Only the ranking rule should be available for this field.

If the user wants to filter on that attribute, he will have to add it in `attributesForFaceting` as well. 

## 3. Future possibilities

- ASC / DESC criterion on string value.
