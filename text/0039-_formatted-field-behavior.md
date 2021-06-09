- Title: _formatted Field Behavior
- Start Date: 2021-05-05
- Specification PR: [#39](https://github.com/meilisearch/specifications/pull/39)
- MeiliSearch Tracking-Issues:

# _formatted Field Behavior

## 1. Functional Specification

### I. Summary

`_formatted` is used in conjunction with raw search results to highlight and/or crop around the query term in the attributes of the document. `_formatted` main goal is to enhance the UI and the UX by providing a nice way to catch the user's eyes on the front-end side while searching.

### II. Motivation

The goal of this specification is to clarify the behavior of `attributesToRetrieve`, `attributesToHighlight` and `attributesToCrop` on `_formatted` response parameter content.

### III. Additional Materials

#### Algolia

By default, Algolia returns `_hightlightResult` even if no `attributesToHighlight` are set at query time. So, by default the value is `*`.
Setting a specific attribute in `attributesToHighlight` will only give this specific attribute in `_highlightResults`.

Unlike highlighting, snippeting must be proactively enabled for each attribute to snippet, however Algolia authorize the usage of `*` to snippet all attributes.
Setting a specific attribute in `attributesToSnippet` will only give this specific attribute `_snippetResult`.

### IV. Explanation

#### Current MeiliSearch Behavior (0.20)

Given a document made of three fields `title`, `actor`, and `poster`.
```
{
    "title": "Prince Avalanche",
    "actor": "Prince",
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

As a user I get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "actor": "Prince",
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

As a user I get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "actor": "Prince",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "Prince Avalanche",
                "actor": "Prince",
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            }
        }
    ]
}
```

The `_formatted` field appears in the search response as soon as the `attributesToHighlight` and/or `attributesToCrop` parameters are sent as a query parameter and are filled with a value representing an existent or inexistent field. Using an inexistent field is similar to setting `attribuesToHighlight`/`attributesToCrop` to `"*"` but the highlight/cropping is not compute in the `_formatted` field. The fields are returned in a raw format.

**Example 3**

Given these search parameters:

```
{
    "q": "Prince",
    "attributesToRetrieve": ["title"],
    "attributesToHighlight": ["actor"]
}
```

As a user I get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "_formatted": {
                "actor": "<em>Prince</em>"
            }
        }
    ]
}
```

`_formatted` is only filled with the fields set in `attributesToHighlight` despite the fact that the user only ask for `title` in `attributesToRetrieve`.

**Example 4**

Given these search parameters:

```
{
    "q": "Prince",
    "attributesToRetrieve": ["actor", "title"],
    "attributesToHighlight": ["actor"]
}
```

As a user I get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "actor": "Prince",
            "_formatted": {
                "actor": "<em>Prince</em>"
            }
        }
    ]
}
```

`_formatted` is only filled with the fields in `attributesToHighlight` despite the fact that the user ask for `actor` and `title` in `attributesToRetrieve`.

**Example 5**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["*"],
    "attributesToHighlight": ["title"]
}
```

As a user I get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "actor": "Prince",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "<em>Prince</em> Avalanche",
                "actor": "Prince",
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
            }
        }
    ]
}
```

`_formatted` field behavior that is supposed to be controlled by `attributesToHighlight` and `attributesToCrop` is dependent of `attributesToRetrieve`.

`_formatted` is filled with all the `attributesToRetrieve` despite the fact that the user only ask for one specific field in `attributesToHighlight` or `attributesToCrop`. The highlighing/cropping is only computed on the targeted field in `attributesToHighlight`. Other fields are returned as raw result in `_formatted`.

**Example 6**

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

`_formatted` is only filled with the `attributesToRetrieve` fields despite the fact that the user may wants all fields to be in `_formatted` and be highlighted or cropped given `attributesToHighlight` or `attributesToCrop` values.

#### Current Transplant/Milli Behavior (0.21)

**Example 1**

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["*"]
}
```

As a user I get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "actor": "Prince",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
        }
    ]
}
```

Since `attributesToHighlight` and `attributesToCrop` are not set, `_formatted` is not computed.

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

As a user I get:

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

Unlike v0.20, if no field are matched to be formatted, `_formatted` is not computed nor returned in response.

**Example 3**

Given these search parameters:

```
{
    "q": "Prince",
    "attributesToRetrieve": ["title"],
    "attributesToHighlight": ["actor"]
}
```

As a user I get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "_formatted": {
                "actor": "<em>Prince</em>"
            }
        }
    ]
}
```

`_formatted` is only filled with the fields set in `attributesToHighlight` despite the fact that the user only ask for `title` in `attributesToRetrieve`.

Same behavior as v0.20 release.

**Example 4**

Given these search paramters:

```
{
    "q": "Prince",
    "attributesToRetrieve": ["actor", "title"],
    "attributesToHighlight": ["actor"]
}
```

As a user i get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "actor": "Prince",
            "_formatted": {
                "actor": "<em>Prince</em>"
            }
        }
    ]
}
```

Same behavior as v0.20 release.

**Example 5**

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
            "actor": "Prince",
            "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg",
            "_formatted": {
                "title": "<em>Prince</em> Avalanche"
            }
        }
    ]
}
```

Unlike v0.20, `_formatted` is only containing fields set in `attributesToHighlight` and `attributesToCrop`. Independently from `attributesToRetrieve` value.

**Example 6**

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
                "actor": "<em>Prince</em>,
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
            }
        }
    ]
}
```

Unlike v0.20, `attributesToHighlight` set fields to be in `_formatted` independently from `attributesToRetrieve`.

#### Expected MeiliSearch Behavior (0.21)

✅ If `attributesToRetrieve` is not set as a parameter, the expected behavior is the same as if `attributesToRetrieve` is equal to `*`.
✅ If `attributesToHighlight` and `attributesToCrop` are not set, do not return `_formatted` and don't compute highlights and crops.
✅ If cumulated fields in `attributesToHighlight` and `attributesToCrop` resolve to only having non-existent fields, do not return `_formatted`.
✅ If `attributesToRetrieve` is equal to `*` and `attributesToHighlight` or `attributesToCrop` are equals to `*`, return `_formatted` and compute highlights and crops on each fields.
✅ If `attributesToRetrieve` is equal to `*` and `attributesToHighlight` or `attributesToCrop` contains a set of fields, return `_formatted` containing fields declared in `attributesToRetrieve` and compute highlights and crops on fields declared in `attributesToHighlight` or `attributesTocCrop`.

**Edge cases**

Given these search parameters:

```
{
    "q": "Prince",
    "attributesToRetrieve": ["title"],
    "attributesToHighlight": ["actor"]
}
```

I want to get:

```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "_formatted": {
                "title": "Prince Avalanche",
                "actor": "<em>Prince</em>"
            }
        }
    ]
}
```

✅ Stay consistent with the fact that every `attributesToRetrieve` are set in `_formatted` result but do not need to be necessary computed for highlighting and cropping until they are declared in `attributesToHighlight` and `attributesToCrop`.

Given these search parameters:
```
{
    "q": "prince",
    "attributesToRetrieve": ["title"],
    "attributesToHighlight": ["*"]
}
```

I want to get:
```
{
    "hits": [
        {
            "title": "Prince Avalanche",
            "_formatted": {
                "title": "<em>Prince</em> Avalanche",
                "actor": "<em>Prince</em>,
                "poster": "https://image.tmdb.org/t/p/w1280/3KHiQt54usbHyIjLIMzaDAoIJNK.jpg"
            }
        }
    ]
}
```

✅ If `attributesToHighlight` or `attributesToCrop` contains a field that is not declared in `attributesToRetrieve`, it his added to `_formatted` and it is highlighted and/or cropped.

### V. Impact on Documentation
N/A

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities

- Add support of `-name_of_attribute` to remove a specific attribute when using `*`. E.g. `"attributesToHighlight": "*, -title"`. All except title. This is really useful when having a lot of attributes in a document. It could work at least for `attributesToRetrieve`, `attributesToHighlight` and, `attributesToCrop`. The usage of these operator will be disjunctive between `attributesToHighlight` and `attributesToCrop` parameters.
- Rename `attributesToRetrieve`
