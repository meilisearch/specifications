- Title: Filter and facet behavior
- Specification PR:

# Facet and Filter Behavior

## 1. Feature Description and Interaction

### Summary

Contrary to MeiliSearch v0.20.0, in the search engine that will be released in the v0.21.0, the users need to declare in `attributesForFaceting` the attributes on which they want to apply the filters.

Why? Under the hood and during the indexation, the core engine creates structures of data that improve the performances drastically during the search with filters.

### Motivation

Because the users need to set the attributes to `attributesForFaceting` no matter what they are using (`filters` or `facetFilters`) during the search, we need to re-define the usage of the filters/facets and stay as ISO as possible with the MeiliSearch v0.20.0.

### Additional Materials

TBD

### Explanation

#### Remove `facetFilters`, keep `filters`

The usage of `facetsFilter` is not needed anymore since everything is doable by only using `filters`.

```json
// Settings
{
    "attributesForFaceting": ["author"]
}
// Search
{
    "q": "",
    "filters": "price < 20",
    "facetFilters": ["author:'JK Rowling'"]
}
```

becomes

```json
// Settings
{
    "attributesForFaceting": ["author", "price"]
}
// Search
{
    "q": "",
    "filters": "price < 20 AND author = 'JK Rowling'"
}
```

⚠️ The operator `:` is not used anymore and is replaced by the `=`.

#### Operator behavior during the search

During the search, the operators should behave as they already do in MeiliSearch v0.20.0.

Here is a quick reminder of the v0.20.0 operator behavior:

- `<`, `=<`, `>`, `=>` => number operation only. MeiliSearch returns only the documents with numbers for this field.
ex: `price > 19` -> does not return `"price": "20"` but returns `"price": 20` -> no document conversions are done.
- `=`, `!=`/`NOT` => string and number operation. MeiliSearch returns the documents with numbers, string (and array of strings) for this field.
- cannot filter on `null`, objects, arrays of "undefined elements" (ex: array of `null`)

#### Accepted syntaxes for `filters`

Two syntaxes will be accepted for the `filters` parameter during the search:

- the string syntax with the `AND`/`OR` operators
ex:
```json
{
    "filters": "(genres = Comedy OR genres = Romance) AND director = 'Mati Diop'"
}
```

- the array syntax:
ex:
```json
{
    "filters": [["genres = Comedy", "genres = Romance"], "director = 'Mati Diop'"]
}
```


#### `filters` and `facetsDistribution` behavior

##### MeiliSearch v0.20.0 with `filters`

In MeiliSeach v0.20.0, with the following documents

```json
[
  { "id": 456,  "genre": "adventure", "price": 12  },
  { "id": 1,    "genre": "fantasy",   "price": 456 }
]
```

...and the following search

```json
{
    "q": "",
    "filters": "price = 12",
    "facetsDistribution": ["genre"]
}
```

...we get the following results:

```json
{
    "hits": [{ "id": 456,  "genre": "adventure", "price": 12 }],
    "facetsDistribution": {
        "genre": {
            "fantasy": 1,
            "adventure": 1
        }
    }
}
```

=> `fantasy` is set to `1` despite the fantasy book does not have a `price` equals to `12` as required in the `filters`.
=> the `filters` and the `facetsDistribution` are not related: the `facetsDistribution` is applied before the filters.

##### MeiliSearch v0.20.0 with `facetFilters`

In MeiliSeach v0.20.0, with the following documents

```json
[
  { "id": 456,  "genre": "adventure", "price": "12"  },
  { "id": 1,    "genre": "fantasy",   "price": "456" }
]
```
(the test is done with `price` as strings because MeiliSearch v0.20.0 cannot facet on numbers)

...and the following search

```json
{
    "q": "",
    "facetFilters": ["price:12"],
    "facetsDistribution": ["genre"]
}
```

...we get the following results:

```json
{
    "hits": [{ "id": 456,  "genre": "adventure", "price": "12" }],
    "facetsDistribution": {
        "genre": {
            "fantasy": 0,
            "adventure": 1
        }
    }
}
```

=> `fantasy` is set to `0` because the fantasy book does not have a `price` equals to `12` as required in the `facetFilters`.
=> the `filters` and the `facetsDistribution` are related: the `facetsDistribution` is applied after the facet filters.

##### Final decision for v0.21.0

MeiliSearch v0.21.0 will behave as the `facetFilters` already behave in MeiliSearch v0.20.0: the `facetsDistribution` will be applied after the filters.

#### TLDR; all the breaking changes

Here is the summary of all the breaking changes (that are detailed in the paragraphs above):

- The `facetFilters` parameter during the search is removed. Only `filters` can be used.
- The users need to set the attributes to `attributesForFaceting` to use the filters during the search via the `filters` parameters.
- The users can now pass an attribute containing numbers (float or integer) in `attributesForFaceting`. It means they can use `filters` and `facetsDistribution` on this numeric field.
- The `filters` parameter can accept both syntax: string (with `OR`/`AND`) and array.
- The `:` operator does not exist anymore (was previously present in `facetFilters` in v0.20.0) and is replaced by `=`.
- The `facetsDistribution` is now applied after the `filters` parameter. This point is currently not documented, not sure this is useful to add it to the docs.
- All the integer in the user documents are converted into float. So we integers with high values lose precision. However, integers from −2^53 to 2^53 (−9007199254740992 to 9007199254740992) can be exactly represented, which is enough in 99% of cases. Not sure this is important to documented it either.

### Impact on Documentation

// TODO

## 2. Technical Specifications

// TODO

### Architecture
### Implementation Details
### Corner Cases

## 3. Future Possibilities

TBD
