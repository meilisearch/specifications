- Title:
- Start Date:
- Specification PR:
- MeiliSearch Issue: 

# Facet Numbers / Facet Range

## 1. Feature Description and Interaction

### Summary

Handle additional operation on Number typed Facets like:
- higher to
- lower to
- equal to
- higher or equal to
- lower or equal to

Handle additional `facetStats` on Number typed Facets like:
- min
- max
- sum (optional?)
- average (optional?)

Provide the user a way to configure it.

### Motivation

As an e-commerce platform, I want to provide a **Price Slider** to be able to filter products on a price range;

As an e-commerce platform, I want to provide a **Customer Reviews Rank** (`★★★☆☆`) to be able to filter products with a minimum rank;

### Additional Materials
### Explanation

To create a **Price Slider** we would need a least:
- higher or equal to
- lower or equal to
to filter on a range of price and:
- min
- max

To provide the minimum and maximum price, the End User can choose.

To create a **Customer Reviews Rank** we would need a least:
- higher or equal to

To filter products with a minimum rank.

### Impact on Documentation

Changes on the `search` route request adding new operators:
- higher to
- lower to
- equal to
- higher or equal to
- lower or equal to


Changes on the `search` route response adding new `facetStats`:
- min
- max
- sum
- average

Changes in `settings` route changing `AttributeForFaceting` to handle Number typed Facets.

## 2. Technical Specifications

### Architecture
### Implementation Details

We have a database for the facets, the keys are prefixed by the field_id (u8), a level (u8) and, the facet value (i64, f64). The facet values don't have a level when those are strings. The data stored under those keys is the document ids that are faceted under those facets values.

The type of the facet (i.e. i64, f64 or string) is stored in another data structure and this is by using it that we know how to read the facet value. If the facet type is a number we are able to use more operators like greater than or lower than (e.g. <, <=, >, >=, =, !=).

#### Indexing phase

When documents come in and fields are declared as facets, we start storing the facet values in the previously described database, the key becomes the facet value (as a globally ordered byte slice) and, the entry data now contains the document id that contains this facet value. Note that if the facet value is a number we store it like [field id][level][left facet value][right facet value] where the level is 0 and if it is a string then we don't store the level.

Once the facet values that are numbers are stored we got a list of facet values prefixed with the field id and the base level (i.e. 0). We use this base level to generate more levels, each level contains groups of 4 groups of the level below, so level 1 aggregates the ids of the documents of each group of 4 facet values of level 0. The left and right facet values are the inclusive bounds of the group, the level 0 group have equal left and right bounds.

#### Querying phase

Those levels are used to reduce the number of entries to run through, reducing the time it takes to answer too wide range filter queries, like duration > 0 where 80% of the entries will match. We go through each of the levels going from the higher one, the one which describes the biggest amount facet values and, we go deeper in the levels to find a better fitting bound.

### Corner Cases

It is hard to detect the type of a `facet` (`float`/`int`/`string`), and we should precise it in `settings`

#### Q&A

Can we count every Number typed facets as `float`? Making the type's detection simpler.
> floats need more space than integers to be stored in meilisearch,
> comparing to integer, there would be significant performance dif,
> there is a precision loss with floats which is not acceptable for certain cases like `timestamps`.

## 3. Future Possibilities

- `sum` and `average` in returned `facetStats`