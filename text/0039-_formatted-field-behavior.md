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

By default, Algolia returns `_hightlightResult` even if no `attributesToHighlight` are set at query time. So, by default the value is `*`.
Setting a specific attribute in `attributesToHighlight` will only give this specific attribute in `_highlightResults`.

Unlike highlighting, snippeting must be proactively enabled for each attribute to snippet, however Algolia authorize the usage of `*` to snippet all attributes.
Setting a specific attribute in `attributesToSnippet` will only give this specific attribute `_snippetResult`.

### IV. Explanation

#### Current MeiliSearch Behavior (0.20)

Given a document made of two fields `title` and `poster`
```
{
    "title": "Prince Avalanche",
    "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
}
```

**Example 1**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["*"]
}
```

As a user i get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
        }
    ]
}
```

Since `attributesToHighlight` and `attributesToCrop` are not set, `_formatted` is not computed.


**Example 2**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["*"],
    "attributesToHighlight": ["wrongFieldName"]
}
```

As a user i get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "Prince Avalanche",
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
            }
        }
    ]
}
```

The `_formatted` field appears in the search response as soon as the `attributesToHighlight` and/or `attributesToCrop` parameters are sent as a query parameter and are filled with a value representing an existent or inexistent field. Using an inexistent field is similar to setting `attribuesToHighlight`/`attributesToCrop` to `"*"`

**Example 3**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["*"],
    "attributesToHighlight": ["title"]
}
```

As a user i get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "<em>Prince</em> Avalanche",
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
            }
        }
    ]
}
```

`_formatted` field behavior that is supposed to be controlled by `attributesToHighlight` and `attributesToCrop` is dependent of `attributesToRetrieve`.

`_formatted` is filled with all the `attributesToRetrieve` despite the fact that the user only ask for one specific field in `attributesToHighlight` or `attributesToCrop`.

**Example 4**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["title"],
    "attributesToHighlight": ["*"]
}
```

As a user i get:
```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "_formatted": {
                "title": "<em>Prince</em> Avalanche"
            }
        }
    ]
}
```

`_formatted` is only filled with the `attributesToRetrieve` fields despite the fact that the user wants all fields to be highlighted or cropped given `attributesToHighlight` or `attributesToCrop` value.

#### Current Transplant/Milli Behavior (0.21)

**Example 1**

Same behavior as v0.20 release.

**Example 2**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["*"],
    "attributesToHighlight": ["wrongFieldName"]
}
```

As a user i get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
        }
    ]
}
```

Unlike v0.20, if no field are matched to be formatted, `_formatted` is not computed.

**Example 3**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["*"],
    "attributesToHighlight": ["title"]
}
```

As a user i get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "<em>Prince</em> Avalanche"
            }
        }
    ]
}
```

Unlike v0.20, `_formatted` is only computed for the fields set in `attributesToHighlight` and `attributesToCrop`. Independently from `attributesToRetrieve` value.

**Example 4**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["title"],
    "attributesToHighlight": ["*"]
}
```

As a user i get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "_formatted": {
                "title": "<em>Prince</em> Avalanche",
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
            }
        }
    ]
}
```

Unlike v0.20, `attributesToHighlight` set fields to be in `_formatted` independently from `attributesToRetrieve`.

#### Expected MeiliSearch Behavior (0.21)
TBD

### V. Impact on Documentation
N/A

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities

- Add support of `-name_of_attribute` to remove a specific attribute when using `*`. E.g. `"attributesToHighlight": "*, -title"`. All except title. This is really useful when having a lot of attributes in a document. It could work at least for `attributesToRetrieve`, `attributesToHighlight` and, `attributesToCrop`. The usage of these operator will be disjunctive between `attributesToHighlight` and `attributesToCrop` parameters.
- Rename `attributesToRetrieve`