- Title: Formatting Search Results
- Start Date: 2021-05-05
- Update Date: 2022-03-01

# Formatting Search Results

## 1. Summary

This specification focuses on the different features of Meilisearch related to the formatting of search results.

## 2. Motivation

At Meilisearch, we strive to facilitate the building of an efficient and enjoyable search experience for all our users.

The formatting of search results exists to emphasize why a document is relevant to a user's search. This intention is achieved by various means, such as highlighting or cropping. This way, an end-user can evaluate the relevance of the search results at a glance.

## 3. Functional Specification

### 3.1. `formatter` API Resource

The formatter API resource allows configuring how the search results are formatted at the search time.

#### 3.1.1. `formatter` object properties

| Field                 | Type            | Required |
|-----------------------|-----------------|----------|
| attributesToHighlight | Array of string | False    |
| highlightPreTag       | String          | False    |
| highlightPostTag      | String          | False    |
| attributesToCrop      | Array of string | False    |
| cropLength            | Integer         | False    |
| cropMarker            | String          | False    |
| matches               | Boolean         | False    |

##### 3.1.1.1. `attributesToHighlight`

- Type: Array of string
- Required: False
- Default: `[]`

Highlights document parts matching query terms in the specified attributes by enclosing them with `highlightPreTag` and `highlightPostTag`.

`attributesToHighlight` only works on values of the following types: `string`, `number`, `array`, `object`.

When this parameter is set, returned search results include a `_formatted` object containing the highlighted terms.

> See _FORMATTED OBJECT DETAILS. TODO

Instead of supplying individual attributes, you can provide `["*"]` as a value: `attributesToHighlight=["*"]`. This will highlight the values of all attributes present in `attributesToRetrieve`.

##### 3.1.1.2. `highlightPreTag`

- Type: String
- Required: False
- Default: `"<em>"`

Specify the tag to put **before** the matched part to highlight.

##### 3.1.1.3. `highlightPostTag`

- Type: String
- Required: False
- Default: `"</em>"`

Specify the tag to put **after** the matched part to highlight.

##### 3.1.1.4. `attributesToCrop`

- Type: Array of string
- Required: False
- Default: `[]`

Crops the selected attributes' values in the returned search results to the length indicated by the `cropLength` parameter.

When this parameter is set, returned search results include a `_formatted` object containing the highlighted terms.

> See _FORMATTED OBJECT DETAILS. TODO

Optionally, you can indicate a custom crop length for any of the listed attributes: `attributesToCrop=["attributeNameA:25", "attributeNameB:150"]`. The custom crop length set in this way has **priority over** the `cropLength` parameter.

Instead of supplying individual attributes, you can provide `["*"]` as a value: `attributesToCrop=["*"]`. This will crop the values of all attributes present in attributesToRetrieve.

!!!REWORK THIS WHOLE PART!!!
Smart crop. Technical Details
Cropping starts at the first occurrence of the search query (TODO changes it by the most relevant matched part). It only keeps `cropLength` words on each side of the first match, rounded to match word boundaries.

If no query word is present in the cropped field, the crop will start from the first word.

cropLength: 10
```json
{
    "id": "50393",
    "title": "Kung Fu Panda Holiday",
    "poster": "https://image.tmdb.org/t/p/w1280/gp18R42TbSUlw9VnXFqyecm52lq.jpg",
    "overview": "The Winter Feast is Po's favorite holiday. Every year he and his father hang decorations, cook together, and serve noodle soup to the villagers. But this year Shifu informs Po that as Dragon Warrior, it is his duty to host the formal Winter Feast at the Jade Palace. Po is caught between his obligations as the Dragon Warrior and his family traditions: between Shifu and Mr. Ping.",
    "release_date": 1290729600,
    "_formatted": {
        "id": {
            "value": "50393"
        },
        "title": {
            "value": "Kung Fu Panda Holiday"
        },
        "poster": {
            "value": "https://image.tmdb.org/t/p/w1280/gp18R42TbSUlw9VnXFqyecm52lq.jpg"
        },
        "overview": {
            "value": "this year Shifu informs"
        },
        "release_date": {
            "value": 1290729600
        }
    }
}
```
##### 3.1.1.5. `cropLength`


##### 3.1.1.6. `cropMarker`

- Type: String
- Required: False
- Default: `"…"` (U+2026)

##### 3.1.1.7. `matches`

- Type: Boolean
- Required: False
- Default: `false`

### 3.2. `_formatted` response object



## 4. Technical Details
N/A

## 5. Future Possibilities
