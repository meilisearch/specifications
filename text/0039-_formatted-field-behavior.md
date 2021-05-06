- Title: _formatted Field Behavior
- Start Date: 2021-05-05
- Specification PR: [#39](https://github.com/meilisearch/specifications/pull/39)
- MeiliSearch Tracking-Issues:

# _formatted Field Behavior

## 1. Functional Specification

### I. Summary

`_formatted`  is used in conjunction with raw search results to highlight and/or crop around the query term in the attributes of the document. `_formatted` main goal is to enhance the UI and the UX by providing a nice way to catch the user's eyes on frontend side while searching.

### II. Motivation

The goal of this specification is to clear the behavior of `attributesToRetrieve`, `attributesToHighlight` and `attributesToCrop` on `_formatted` response parameter content.

### III. Additional Materials

#### Algolia

By default, Algolia returns `_hightlightResult` even if no `attributesToHighlight` are set at query time. So, by default the value is `'*'`.
Setting a specific attribute in `attributesToHighlight` will only give this specific attribute in `_highlightResults`.

Unlike highlighting, snippeting must be proactively enabled for each attribute to snippet, however Algolia authorize the usage of `*` to snippet all attributes.
Setting a specific attribute in `attributesToSnippet` will only give this specific attribute `_snippetResult`.

### IV. Explanation

#### Current MeiliSearch Behavior (0.20)

The `_formatted` field appears in the search response as soon as the `attributesToHighlight` and/or `attributesToCrop` parameters are sent as a query parameter and are filled with a parameter or as an empty parameter.

`attributesToRetrieve` is independent of `_formatted` and so of `attributesToHighlight` and `attributesToCrop`.

`attributesToHighlight` and `attributesToCrop` have an unwanted behavior on `_formatted` result:

Given a document made of two fields `title` and `poster`
```json
{
    "title": "Prince Avalance",
    "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
}
```

Given these search parameters:
```json
{
    "q": "prince",
    "attributesToHighlight": ["title"]
}
```

As a user i expect to have this response:

```json
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "<em>Prince</em> Avalance"
            }
        }
    ]
}
```

but i get:
```json
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "<em>Prince</em> Avalance",
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
            }
        }
    ]
}
```

`_formatted` is filled with all the `displayedAttributes` despite the fact that the user only ask for one specific field.

This behaviour will be fixed in the `0.21` release in order to only get the fields set in `attributesToHighlight` and `attributesToCrop` in `_formatted`.

#### MeiliSearch Behavior (0.21)

As in the `0.20`, the `_formatted` fields appears in the search response as soon as the `attributesToHighlight` and/or `attributesToCrop` parameters are sent with an empty value.

E.g. `"attributesToHighlight": `

As in the `0.20`, `_formatted` is also filled with all the `displayedAttributes` despite the fact that the user only ask for one specific field in `attributesForHighlight` or `attributesToCrop`.

The `_formatted` field appears in the search response as soon as the `attributesToHighlight` and/or `attributesToCrop` parameters are sent as a query parameter and filled with an existent parameter.

The next rules clarify the behavior of `_formatted` parameter given `attributesToHighlight` and `attributesToCrop` states and combination.

✅  `_formatted` response parameter **SHOULD NOT** be returned in the given cases:

- `attributesToHighlight` is set but is empty AND `attributesToCrop` is set but is empty.
- `attributesToHighlight` is set but is empty AND `attributesToCrop` isn't set.
- `attributesToHighlight` is set but only contains a non-existent field AND `attributesToCrop` is set but only contains a non-existent field.
- `attributesToHighlight` is set but only contains a non-existent field AND (`attributesToCrop` is not set OR `attributesToCrop` is set but is empty).

✅ `_formatted` response parameter **SHOULD** be returned in any cases of:

- `attributesToHighlight` is set with `*`. When `attributesToHighlight` is not set, the engine will use `*` as default parameter. So each `displayedAttributes` will be in the `_formated` result in that specific case.
- `attributesToCrop` is set with `*`.
- `attributesToHighlight` is set with **1..N** existing attributes.
- `attributesToCrop` is set with **1..N** existing attributes.


> ⚠️ Note that if one of the two parameters is requested but empty or contains an attribute that does not exist but the other is valid, `_formatted` will be returned based on the valid parameter.

> ⚠️ Note that `attributesToRetrieve` is independent of `_formatted` and so of `attributesToHighlight` and `attributesToCrop`.

> ⚠️ Note that `displayedAttributes` will control what will be displayed in search results despite asking for fields to be retrieved with `attributesToRetrieve` or to be formatted with `attributesToHighlight` or `attributesToCrop`.


### V. Impact on Documentation
N/A

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities

- Add support of `-name_of_attribute` to remove a specific attribute when using `*`. E.g. `"attributesToHighlight": "*, -title"`. All except title. This is really useful when having a lot of attributes in a document. It could work at least for `attributesToRetrieve`, `attributesToHighlight` and, `attributesToCrop`. The usage of these operator will be disjunctive between `attributesToHighlight` and `attributesToCrop` parameters.
- Rename `attributesToRetrieve`