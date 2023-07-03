# Ranking Score

## 1. Summary

Adds two kinds of scores to documents returned by a [search query](./0118-search-api.md).

## 2. Motivation

When configuring the Meilisearch relevancy according to their needs, users cannot know why one document has been favored over another.

Showing how the documents ranked according to Meilisearch’s ranking rules unlocks:

- Further customization of the developer workflow, such as fine-tuning settings and improving relevancy for example.
- Returning a unified list of results for multi-index search queries
- Sharding
- Debugging and helping users better understand how ranking works

## 3. Functional Specification

### 3.1. Ranking score

A ranking score is a number attached to each document returned by a search when the [`showRankingScore`](./0118-search-api.md#3117-showrankingscore) flag is set to true in the search query.

#### 3.1.1. Scale and interpretation

The ranking score is contained between 1.0 and 0.0. A higher score signifies better relevancy, with 1.0 representing a perfect match, and 0.0 indicating that the document does not match the query (Meilisearch should not return documents that do not match the query).

That number rates the relevancy of the document with respect to the specified search query and the current settings of the index.

The score of a document follows its relevancy in the sense of Meilisearch, in that the first few ranking rules have a much higher influence on the score than the next rules. This is consistent with the way that later ranking rules are only used to break ties with earlier ranking rules, when ranking documents.

#### 3.1.2. Score independence

The score of a document is independent of what other documents are contained in the index but is influenced by the settings of the index. The table below details all the settings that can influence the score. Unlisted settings do not influence the ranking score.

| Setting name | Influences if | Rationale |
|--------------|---------------|-----------|
|`searchableAttributes`|The `attribute` ranking rule is used|The `attribute` ranking rule rates the document depending on the attribute in which the query terms show up. The order is determined by `searchableAttributes`|
|`rankingRules`|Always|The score is computed by computing the subscore of each ranking rule with a weight that depends on their order.|
|`stopWords`|Always|Stop words influence the `words` ranking rule, which is almost always used|
|`synonyms`|Always|Synonyms influence the `words` ranking rule, which is almost always used|
|`typoTolerance`|The `typo` ranking rule is used|Used to compute the maximum number of typos for a query|


Additionally, the following can impact score independence:

- if the `attribute` ranking rule is used, but `searchableAttributes` has not been specified, then the score is dependent on all the fields that appear in documents and their precise order, as determined by Meilisearch.
the score is dependent on the search query.

Depending on the use case, it can be meaningful to compare scores coming from indexes with settings that are different:

- When comparing two scores produced on two indexes with different settings, possibly on a distinct search query, one is comparing the relevancy of each of the scored documents to their respective search query. This is good to present the most relevant documents first when working with heterogeneous indexes, without taking into account which document best suits one single query.
- On the other hand, to find what document best suits one single query against two homogeneous indexes, one must be careful to make sure that the indexes have the settings above set to the same value.

#### 3.1.3. The sort ranking rules do not impact the score

Custom `sort` and `geosort` ranking rules modify the ranking of documents such that they are returned sorted by the value of the target field, rather than by their relevancy to the search query.

As such, these ranking rules have no impact on the score. As a corollary of this, if a `sort` ranking rule is not the last ranking rule, then it is possible to see documents returned with ranking scores that are not monotonically decreasing.

Similarly, re-ranking documents by their ranking score will ignore any `sort` ranking rule.

If you need to factor sort ranking rules into your score, then use the [ranking score details](#32-ranking-score-details).

### 3.2. Ranking score details

(EXPERIMENTAL) The ranking score details are represented as an object attached to each document returned by a search when the [`showRankingScoreDetails`](./0118-search-api.md#3118-showrankingscoredetails) flag is set to true in the search query.

The ranking score details are experimental and require enabling the corresponding [experimental feature](./0193-experimental-features.md#score-details).

#### 3.2.1. General shape

The fields of the object have for key the identifier of the various ranking rules that were applied, and for value an object with at least the following field:

- `order`: the numerical order in which the ranking rule was applied. Starts at 0. Consecutive numbers denote ranking rules consecutively applied.

Additionally, all ranking rules except the `sort` and `geosort` ranking rules have the following field:

- `score`: the relevancy score of the document relative to this search query, for this ranking rule. A number between 1.0 and 0.0, with 1.0 meaning a perfect match to the query according to the ranking rule, and 0.0 no match.

#### 3.2.2. Ranking-rule-specific fields

Each ranking rule exposes specific fields meant to provide semantic information about how the ranking rule was applied to the document.

The table below details these rule-specific fields.

​
| Ranking rule           | Field description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `words`                | <ul><li>`matchingWords`: Number of words in the query that match in the document. The higher the better</li><li>`maxMatchingWords`: Maximum number of words in the query that can match in the document for this iteration of the `words` ranking rule. Usually, the query length, but if one of the query terms is set as a stop word, it won’t be counted here.</li></ul>                                                                                                                    |
| `typo`                 | <ul><li>`typoCount`: Number of typos to correct in the query so that the document matches for this iteration of the `typo` ranking rule.</li><li>`maxTypoCount`: Maximum number of typos possible in a document for this iteration of the `typo` ranking rule.</li></ul>                    |
| `proximity`            | No rule-specific field                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `attribute`            | <ul><li>`attribute_ranking_order_score`: Results sorted based on the attribute ranking order</li><li>`query_word_distance_score`: Documents with attributes containing the query words close to their position in the query will be considered more relevant than documents containing the query words far from their position in the query</li></ul>                                                    |
| `exactness`            | <ul><li>`matchType`: It has one of the following values:<ul><li>`exactMatch`: The query exactly matches the entire value of an attribute</li><li>`matchesStart`: The query matches exactly the start of the value of an attribute</li><li>`noExactMatch`: The query doesn't exactly match a document </li></ul></li><li>`matchingWords`: for `matchesStart`, the number of exact words contained in an attribute. The higher the better</li><li>`maxMatchingWords`: for `noExactMatch`, the maximum number of exact words contained in an attribute</li></ul> |

#### 3.2.3. Sort ranking rules

`Sort` and `geosort` ranking rules appear as fields in the score details, but with the following difference:

- Their key follows the following format: `{:attribute-sorted-on}:{:sort-direction}`, with the `:attribute-sorted-on` the name of the attribute that is being sorted on, and the `:sort-direction` either `asc` if the sort is in ascending order, or `desc` if the sort is in descending order. For the `geosort` ranking rule, it is similarly `_geoPoint({:lat}, {:lng}):{:sort-direction}`, with the `:lat` and `:lng` being the latitude and resp. longitude of the point that serves as base to sort by distance.
- They don't have a `score` field, but instead they have a `value` field, representing the value used to sort the document. It is typically the value of the sorted attribute for the document, but can sometimes be a subvalue (case where the value is an array of values).
- For the `geosort`, there is an additional `distance` field representing the distance between the target point and the point used in the document to sort the document.

#### 3.2.4 Example

The following is an example of a `_scoreDetails` returned for a document matching a search query.


```json
"_rankingScoreDetails": {
    "words": {
        "order": 0,
        "matchingWords": 1,
        "maxMatchingWords": 1,
        "score": 1
    },
    "typo": {
        "order": 1,
        "typoCount": 0,
        "maxTypoCount": 1,
        "score": 1
    },
    "proximity": {
        "order": 2,
        "score": 1
    },
    "attribute": {
        "order": 3,
        "attributes_ranking_order": 0.8333333333333334,
        "attributes_query_word_order": 1,
        "score": 0.8333333333333334
    },
    "exactness": {
        "order": 4,
        "matchType": "exactMatch",
        "score": 1
    },
    "release_date:asc": {
        "order": 5,x
        "value": 1165881600
    }
}
```

## 4. Technical Details

### 4.1. Ranking score calculation

The ranking score calculation in this section is given for informative purposes and is not normative.

The implementation computes the [ranking score](#31-ranking-score) from each ranking rule (excluding `sort` and `geosort`) with two bits of data per ranking rule. For the `k`th applied ranking rule:

- The maximum rank `max_rank_k` that a document can score with the rule, [independently from the other documents in the index](#312-score-independence)
- The rank `rank_k` of that document for that rule, with the highest rank being equal to the maximum rank, and the lowest rank being equal to 1.

The score is given by the following formula, assuming `n` ranking rules denoted from `0` to `n-1`:

```
score = sum(i in 0..(n-1), (rank_i - 1) / product(j in 0..=i, max_rank_j)) + (rank_(n-1) / product(i in 0..n, max_rank_i))
```

The intuition behind this formula is that every document falls in a range for each rule, between `rank_i / max_rank_i` and `(rank_i - 1) / max_rank_i`, and the next ranking rule allows to refine where the document is in this range, with the last ranking rule providing the exact score.


### 4.2. Hidden ranking rules

If the [`displayedAttributes`](./0123-displayed-attributes-setting-api.md) list is defined, then attributes that are not part of that list, but are used in `sort` ranking rules are **hidden**.

Instead of seeing `{:attribute-sorted-on}:{:sort-direction}` like described in [the relevant section](#323-sort-ranking-rules), the name of that field is replaced with `<hidden-rule-{:number}>`, with `{:number}` a number that serves to uniquely distinguish between such hidden rules.

Note: that number is not guaranteed to start at 0 nor to be consecutive. The only guarantee is that no hidden ranking rule will have the same number.

Furthermore, the `value` that was used to sort the document is also hidden and replaced by `"<hidden>"`.

### 4.3. Disabled optimization

The engine optimizes search by skipping the application of ranking rules when there's only one remaining document (no tie to break).

To compute an accurate score, however, all ranking rules must be applied, so this optimization is disabled as soon as a score is requested in the search request. When no score are requested, the optimization is active.

## 5. Future Possibilities

- Extend the [multi-search API](./0192-multi-search-api.md) to rerank documents according to their score, providing federated search.
