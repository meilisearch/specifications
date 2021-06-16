- Title: Exactness Criterion
- Start Date: 2021-04-22
- Specification PR: [#36](https://github.com/meilisearch/specifications/pull/36)
- MeiliSearch Tracking-Issues: [milli/#165](https://github.com/meilisearch/milli/issues/165)

# Exactness Criterion

## 1. Functional Specification

### I. Summary

Exactness Criterion is used within ranking rules to sort results by the similarity of the matched words with the query words. Documents that contain exactly the same terms as the ones queried first are placed first.

> Ranking rules are built-in rules that ensure relevancy in search results. Ranking rules are applied in a default order which can be changed in the settings. You can add or remove rules and change their order.

### II. Motivation

While we continue to advance the new Milli engine towards a version offering the same features as Meilisearch.

This specification aims to change the behavior of the `exactness` criterion to enhance the relevanvy of search results.

### III. Additional Materials

#### TypeSense

Typesense calculates a `_text_match` score for ranking the documents about text relevance.

`_text_match` score is computed from different criterias like frequency, edit distance, proximity and, ordering of `query_by` fields. [See more details here](https://typesense.org/docs/0.19.0/guide/ranking-and-relevance.html#text-match-score).

If documents have the exact same `_text_match` score, TypeSense accepts two more numerical fields in the `sort_by` parameter to break the tie.

E.g. `sort_by=_text_match:desc,price:desc,rating:desc`

It is possible to put a numerical field before the `_text_match` score.

E.g. `sort_by=price:desc,_text_match:desc`

`sort_by` parameter is set at query time.

#### Algolia

Algolia explains the Exact criterion like:

> Records with words that match query terms exactly are ranked higher. The more matching words in a record’s attribute, the higher a record is ranked. By default, an exact match occurs when a full word in a query is matched without typos to a word in an attribute. An inexact match is one that has typos or only matches on a prefix.

> Additionally, synonym matching and plural/singular matching are considered exact. Thus, a word is considered an exact match if its synonym matches a query exactly.

Adding some exceptions by default:

[...] single-word matches on multi-word attributes aren’t considered exact matches by default [...]
[...] Multiword synonyms are not counted as exact matches by default [...]

It is possible to specify that multi-words synonyms are considered an exact match using `alternativesAsExact` setting. Algolia also permit to disable some attributes to be used for exact criterion with `disableExactOnAttributes`setting.

### IV. Explanation

#### Current MeiliSearch Behavior (0.20)

For each word of the query we match documents:

- without typo
- without prefix
- without ngrams
- without multi-words synonyms
- without word split

> If the query contains only one word, only attribute containing this one word value can match.

##### Pros & Cons

This behavior of the Exact criterion gives a better rank to documents that match the words of the original query, ranking ngrams, complex synonyms, and word split after.

In the specific case of a query containing only 1 word, only 1-word attributes matching will boost documents that have an attribute that exactly contains the query word.

This is useful when an ID is searched but in others cases it could impact the relevancy of search results.

Moreover, when users search with a multi-words query, attributes that exactly contains the query are not boosted.

Because MeiliSearch is a search as you type engine, attributes that exactly begin by the original query should also be boosted.

#### Milli Behavior (0.21)

We will divide Exactness into three layers of ranking.

##### Layer 1: Try to match exactly the query in an attribute

Any document that has an attribute that contains exactly the query:

- without typo
- without prefix
- without ngrams
- without multi-words synonyms
- without word split
- with words in the right order

has a rank of `0` (perfect match)

##### Layer 2: Try to match exactly the query at the start of an attribute

Any document that has an attribute that contains the query at the n firsts positions:

- without typo
- without prefix
- without ngrams
- without multi-words synonyms
- without word split
- with words in the right order

has a rank of `1` (the user didn't finish to type the search but the final query could possibly be an exact match)

##### Layer 3: Try to match exactly the word of the query anywhere in the document

Any other document has a rank of `2 + 1` for each word that is not an exact match.

A word is considered as an exact match when it matches:

- without typo
- without prefix
- without ngrams
- without multi-words synonyms
- without word split

> Given theses layered operations, the higher the rank of a document is, the less relevant it is.

### V. Impact on Documentation

N/A

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities

- In the second layer, we can rank `Any document that has an attribute that contains exactly the query` with the last word considered as prefix.
- Give the user the possibility to deactivate some fields for the Exactness criterion.
- Rename the Exactness criterion.
