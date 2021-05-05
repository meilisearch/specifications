- Title: _formatted Field Behavior
- Start Date: 2021-05-05
- Specification PR: [#]()
- MeiliSearch Tracking-Issues:

# _formatted Field Behavior

## 1. Functional Specification

### I. Summary

### II. Motivation

### III. Additional Materials

#### Algolia

By default, Algolia returns `_hightlighResults` even if no `attributesToHighlight` are set at query time.

### IV. Explanation

#### Current Milli Behavior (0.21)

The `_formatted` field appears in the search response as soon as the `attributesToHighlight` and/or `attributesToCrop` parameters are sent as a query parameter whether or not they are filled.

The next rules clarify the behavior of _formatted parameter given `attributesToHighlight` and `attributesToCrop` states and combination.

✅  `_formatted` response parameter **SHOULD NOT** be returned in the case of:

- `attributesToHighlight` is set but is empty and `attributesToCrop` is set but is empty.
- `attributesToHighlight` is not set and `attributesToCrop` is set but is empty.
- `attributesToHighlight` is set but is empty and `attributesToCrop` isn't set.
- `attributesToHighlight` is set but contains a non-existent field and `attributesToCrop` is set but contains a non-existent field.
- `attributesToHighlight` is set but contains a non-existent field and `attributesToCrop` is not set.
- `attributesToHighlight` is not set and `attributesToCrop` is set but contains a non-existent field.

✅ `_formatted` response parameter **SHOULD** be returned in the case of:

- `attributesToHighlight` is set with `*`.
- `attributesToCrop` is set with `*`.
- `attributesToHighlight` is set with 1..N existing attributes.
- `attributesToCrop` is set with 1..N existing attributes.

> ⚠️ Note that if one of the two parameters is requested but empty or contains an attribute that does not exist but the other is valid, _formatted will be returned based on the valid parameter.

> ⚠️ Note that `attributesToRetrieve` is independent of `_formatted` and so of `attributesToHighlight` and `attributesToCrop`.
>
> ⚠️ Note that `displayedAttributes` will control what will be displayed in search results despite asking for fields to be retrieved with `attributesToRetrieve` or to be formatted with `attributesToHighlight` or `attributesToCrop` if they are not in `displayedAttributes`.

### V. Impact on Documentation
N/A

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities