- Title:
- Start Date:
- Specification PR:
- MeiliSearch Issue: 

# Facet Numbers / Facet Range

## 1. Feature Description and Interaction

### Summary

Handle additonal operation on Number typed Facets like:
- higher to
- lower to
- equal to
- higher or equal to
- lower or equal to

Handle additonal `facetStats` on Number typed Facets like:
- min
- max
- sum (optional?)
- average (optional?)

Provide to user a way to configure it.

### Motivation

As an e-commerce platform I want to provide a **Price Slider** to be able to filter products on a price range;

As an e-commerce platform I want to provide a **Customer Reviews Rank** (`★★★☆☆`) to be able to filter products with a minimum rank;

### Additional Materials
### Explanation

To create a **Price Slider** we would need a least:
- higher or equal to
- lower or equal to
to filter on a range of price and:
- min
- max
to provide the minimum and maximum price the End User can choose.

To create a **Customer Reviews Rank** we would need a least:
- higher or equal to
to filter products with a minimum rank.

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

Changes on `settings` route changing `AttributeForFaceting` to handle Number typed Facets.

## 2. Technical Specifications

### Architecture
### Implementation Details
### Corner Cases

It is hard to detect the type of a `facet` (`float`/`int`/`string`) and we should precise it in `settings`

#### Q&A

Can we count every Number typed facets as `float`? Making the type detection simplier.
> floats need more space than integers to be stored in meilisearch,
> comparing to integer their would be significant performance dif,
> their is a precision loss with floats which is not acceptable for certain cases like `timestamps`

## 3. Future Possibilities

- `sum` and `average` in returned `facetStats`