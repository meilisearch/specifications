- Title: Filter and Facet Behavior
- Specification PR: [#27](https://github.com/meilisearch/specifications/pull/27)
- MeiliSearch Tracking-Issues:

# Facet and Filter Behavior

## 1. Functional Specification

### I. Summary

With v0.21.0, we are trying to erase the distinction between facets and filters. facetFilters is removed as a query parameter. Instead, all filters are performed with the filters parameter. In addition, any attribute you wish to use with filters must first be added to attributesForFaceting (rename pending).

### II. Motivation

Because the users need to set the attributes to `attributesForFaceting` no matter what they are using (`filters` or `facetFilters`) during the search, we need to re-define the usage of the filters/facets and stay as backwards compatible as possible with the MeiliSearch v0.20.0.

### III. Additional Materials

N/A

### IV. Explanation

#### Remove `facetFilters`, keep `filters`

The usage of `facetFilters` is not needed anymore since everything is doable by only using the `filters` parameter.

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

We decided to replace the `:` operator in favor of `=`.

#### Operator behavior during search

During search, logical operators should behave as they already do in MeiliSearch v0.20.0.

Here is a quick reminder of the v0.20.0 operator behavior:

- `<`, `=<`, `>`, `=>` => only operate on number values. MeiliSearch returns only documents that have numbers in this field.
ex: `price > 19` does not return `"price": "20"` but returns `"price": 20` -> no type conversions are done.
- `=`, `!=`/`NOT` => operate on string and number values. MeiliSearch returns only documents that have numbers, strings, or arrays of strings in this field.
- cannot filter on `null`, objects, arrays of "undefined elements" (ex: array of `null`)

#### Accepted syntaxes for `filters`

Three syntaxes will be accepted for the `filters` parameter during search. `String syntax`, `Array syntax` and `Mixed syntax`. 

##### String syntax

The string syntax uses the `AND`/`OR`/`NOT` operators combined with parentheses to express a search filter.

Example:
```json
{
    "filters": "(genres = Comedy OR genres = Romance) AND director = 'Mati Diop'"
}
```

##### Array syntax

The array syntax uses dimensional array to express logical connectives.

- Inner arrays elements are connected by an OR operator (e.g. [["genres:Comedy", "genres:Romance"]]).
- Outer arrays elements are connected by an AND operator (e.g. ["genres:Romance", "director:Mati Diop"]).

Example:
```json
{
    "filters": [["genres = Comedy", "genres = Romance"], "director = 'Mati Diop'"]
}
```

> The array syntax is limited to express a

##### Mixed syntax

The mixed syntax can mix string and array syntaxes.

Let's say that we want to translate
```json
{
    "filters": "((genres = Comedy AND genres = Romance) OR genres = Action) AND director != 'Mati Diop'"
}
```
Example:
```json
{
    "filters": [["genres = Comedy AND genres = Romance", "genres = Action"], "NOT director = comedy"]
}

> Note that string values that are longer than a single word need to be enclosed by quote. `"director = Mati Diop"` will lead to a parsing error. The valid syntax is `"director = 'Mati Diop'"`.

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

In MeiliSearch v0.21.0, `facetsDistribution` will behave with `filters` the same way it currently does with `facetFilters`: the `facetsDistribution` will be applied after the filters.

#### TLDR; all the breaking changes

Here is the summary of all the breaking changes (that are detailed in the paragraphs above):

- The `facetFilters` parameter during the search is removed. Only `filters` can be used.
- The users need to set the attributes to `attributesForFaceting` to use the filters during the search via the `filters` parameters.
- The users can now pass an attribute containing numbers (float or integer) in `attributesForFaceting`. It means they can use `filters` and `facetsDistribution` on this numeric field.
- The `filters` parameter can accept both syntax: string (with `OR`/`AND`/`NOT`) and array.
- The `:` operator does not exist anymore (was previously present in `facetFilters` in v0.20.0) and is replaced by `=`.
- The `facetsDistribution` is now applied after the `filters` parameter. This point is currently not documented, not sure this is useful to add it to the docs.
- All the integer in the user documents are converted into float. So integers with high values lose precision. However, integers from −2^53 to 2^53 (−9007199254740992 to 9007199254740992) can be exactly represented, which is enough in 99% of cases. Not sure this is important to documented it either.

### V. Impact on Documentation

See the previous part.

The documentation should present a complex filter query with multiple levels to precise that MeiliSearch is not limited to deepness level.

Example: 

```json
{
    "filters": "((genres = Comedy OR genres = Romance) AND (director = 'Mati Diop' OR director = 'Wong Kar-wai')) OR genres = 'Fantasy'"
}
```

## 2. Technical Aspects

### I. Abstract

Internal MeiliSearch engine uses an index data structure for each type encountered in an attribute. Doing this eliminate the need for the user to strictly type the attribute and permits to deal with loosely typed document at indexing time. Thus, facilitating the developper experience.

As for example, let’s imagine that we want to index two documents with a price attribute. One containing `"price": "20"` and the second `"price": 20` as value. The two attribute values will be stored in two different indexes.

Using `=` or `!=` operator on `price` attribute will lead the engine to query the two indexes and get matching document ids with an `UNION` operation. So it will return documents matching `"20"` or `20` as values for the `price` attribute.

#### Implementation Details

We have a database for the facets, the keys are prefixed by the field_id (u8), a level (u8) and, the facet value (f64). The facet values don't have a level when the type is a string. The data stored under those keys is the document ids that are faceted under those facets values.

The type of the facet (i.e. f64 or string) is stored in another data structure and this is by using it that we know how to read the facet value. If the facet type is a number we are able to use more operators like greater than or lower than (e.g. <, <=, >, >=, =, !=).

##### Indexing phase

When documents come in and fields are declared as facets, we start storing the facet values in the previously described database, the key becomes the facet value (as a globally ordered byte slice) and, the entry data now contains the document id that contains this facet value. Note that if the facet value is a number we store it like `[field id][level][left facet value][right facet value]` where the level is 0 and if it is a string then we don't store the level.

Once the facet values that are numbers are stored we got a list of facet values prefixed with the field id and the base level (i.e. 0). We use this base level to generate more levels, each level contains groups of 4 groups of the level below, so level 1 aggregates the ids of the documents of each group of 4 facet values of level 0. The left and right facet values are the inclusive bounds of the group, the level 0 group have equal left and right bounds.

##### Querying phase

Those levels are used to reduce the number of entries to run through, reducing the time it takes to answer too wide range filter queries, like duration > 0 where 80% of the entries will match. We go through each of the levels going from the higher one, the one which describes the biggest amount of facet values and, we go deeper in the levels to find better fitting bounds.

### II. Issues Summary

In Milli:
- https://github.com/meilisearch/milli/issues/152: the main issue gathering all the internal changes.

In Transplant:
- https://github.com/meilisearch/transplant/issues/140: expose both syntaxes for `filters`.
- https://github.com/meilisearch/transplant/issues/70: the push of `attributesForFaceting`. The behavior does not change between v0.20.0 and v0.21.0 but it still needs changes in Transplant.
- https://github.com/meilisearch/transplant/issues/81: only keeping `filters` and removing `facetFilters`.

## 3. Future Possibilities

- Provide facet statistics like `min` `max` `sum` `average`
- Rename `attributesForFaceting` for clarity.
