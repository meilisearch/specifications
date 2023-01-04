- Title: Geosearch
- Start Date: 2021-08-02
- Specification PR: [#59](https://github.com/meilisearch/specifications/pull/59)
- Discovery Issue: [#42](https://github.com/meilisearch/product/issues/42)

# Geosearch

## 1. Functional Specification

### I. Summary

The purpose of this specification is to add a first iteration of the **geosearch** feature to give geo-filtering and geosorting capabilities at search time.

#### Summary Key points

- Documents MUST have a `_geo` reserved object to be geosearchable.
- Filter documents by a given geo radius using the built-in filter `_geoRadius({lat}, {lng}, {distance_in_meters})`. It is possible to cumulate several geosearch filters within the `filter` field.
- Sort documents in ascending/descending order around a geo point. e.g. `_geoPoint({lat}, {lng}):asc`.
- It is possible to filter and/or sort by geographical criteria of the user's choice.
- `_geo` must be set as a filterable attribute to use geo filtering capabilities.
- `_geo` must be set as a sortable attribute to use geo sort capabilities.
- There is no `geo` ranking rule that can be manipulated by the user. This one is automatically integrated in the ranking rule `sort` by default and activated by sorting using the `_geoPoint({lat}, {lng})` built-in sort rule.
- Using `_geoPoint({lat}, {lng})` in the `sort` parameter at search leads the engine to return a `_geoDistance` within the search results. This field represents the distance in meters of the document from the specified `_geoPoint`.
- Add an `invalid_geo_field` error.
- Add an alternative message for `invalid_sort` and `invalid_filter` error to handle reserved keywords.
- `invalid_criterion` is renamed to `invalid_ranking_rule` and add an alternative message to handle reserved keywords.

### II. Motivation

According to our user feedback, the lack of a geosearch feature is mentioned as one of the biggest deal-breakers for choosing MeiliSearch as a search engine. A search engine must offer this feature. Some use cases specifically require integrated geosearch capabilities. Moreover, a lot of direct competitors offer it. Users today must find workarounds like using geohash to be able to geosearch documents. We hope to better serve the needs of users by implementing this feature. It allows multiplying the use-cases to which MeiliSearch can respond.

### III. Technical Explanations

#### **As a developer, I want to add geospatial coordinates to a document so that the document can be geosearchable.**

- Introduce a reserved field `_geo` for documents to store geo spatial data from an **object** made of `lat` and `lng` fields for a **JSON format**.
- Introduce a reserved column `_geo` for documents to store geo spatial data from a **string** made of `lat,lng` for a **CSV format**.

##### **JSON Format**

**`_geo` field definition**

- Name: `_geo`
- Type: Object
- Format: `{lat:number|string, lng:number|string}`
- Not required

> 💡 if `_geo` is found in the document payload, `lat` and `lng` are required.
> 💡 `lat` and `lng` must be of float value.
> 💡 `lat` and `lng` field type can be mixed. e.g. `lat` can be a string while `lng` is a number in the same `_geo` object.

##### **CSV Format**

Following the format already defined in the https://github.com/meilisearch/specifications/pull/28/files specification for document indexing from a CSV format. A reserved column `_geo` can be added to specify the geographical coordinates of a document.

csv format example
```
"id:number","label","brand","_geo"
"1","F40","Ferrari","48.862725,2.287592"
```

**`_geo` column definition**

- Name: `_geo`
- Type: String
- Format: `"lat:float,lng:float"`
- Not required

#### POST Add or replace documents `/indexes/{indexUid}/documents`

##### Request body
```json
[
    {
        "id": 1,
        "label": "F40",
        "brand": "Ferrari",
        "_geo": {
            "lat": 48.862725,
            "lng": 2.287592
        }
    }
]
```

##### 202 Accepted - Response body

```json
{
    "updateId": 1
}
```

#### PUT Add or replace documents `/indexes/{indexUid}/documents`

##### Request body

```json
[
    {
        "id": 1,
        "brand": "F40 LM",
        "brand": "Ferrari",
        "_geo": {
            "lat": "48.862725",
            "lng": "2.287592"
        }
    }
]
```

##### 202 Accepted - Response body

```json
{
    "updateId": 2
}
```

- 🔴 Giving a bad formed `_geo` that do not conform to the format causes the `task` payload to fail and returns an [invalid_document_geo_field](0061-error-format-and-definitions.md#invalid_document_geo_field) error.

---

### **As an end-user, I want to filter documents within a geo radius.**

- Introduce a `_geoRadius({lat}, {lng}, {distance_in_meters})` built-in filter rule  to `filter` documents in a geo radius.shape.

**`_geoRadius` built-in filter rule definition**

- Name: `_geoRadius`
- Signature: ({lat:float}:required, {lng:float}:required, {distance_in_meters:int}:required)
- Not required
- `distance_in_meters` only accepts positive value.

>  The `_geo` field has to be set in `filterableAttributes` setting by the developer to activate geo filtering capabilities at search.

#### GET Search `/indexes/{indexUid}/search`

```
...&filter="brand=Mercedes AND _geoRadius(48.862725, 2.287592, 2000)"`
```

#### POST Search `/indexes/{indexUid}/search`

```json
{
    "filter": ["brand = Ferrari", "_geoRadius(48.862725, 2.287592, 2000)"]
}
```

- 🔴 Specifying parameters that do not conform to the `_geoRadius` signature causes the API to return an [invalid_search_parameter_filter](0061-error-format-and-definitions.md#invalid_search_parameter_filter) error.
- 🔴 Using `_geoDistance`, `_geo` or `_geoPoint` in a filter expression causes the API to return an [invalid_search_parameter_filter](0061-error-format-and-definitions.md#invalid_search_parameter_filter) error.

---

### **As an end-user, I want to sort documents around a geo point.**

- Introduce a `_geoPoint({lat}, {lng})` function parameter to `sort` documents around a central point.

**`_geoPoint` built-in sort rule definition**

- Name: `_geoPoint`
- Signature: ({lat:float}:required, {lng:float}:required)
- Not required

Following the [`sort` specification feature](https://github.com/meilisearch/specifications/pull/55):
> The `_geo` field has to be set in `sortableAttributes` setting by the developer to activate geo sorting capabilities at search.
>
>There is no `geo` ranking rule as such. It is in fact within the `sort` ranking rule in an obfuscated way.
>
>`_geoPoint` built-in sort rule can sort documents in ascending/descending order.

#### GET Search `/indexes/{indexUid}/search`

```
    ...&sort=_geoPoint({lat, lng}):asc,price:desc
```

#### POST Search `/indexes/{indexUid}/search`

```json
{
    "sort": "_geoPoint({lat, lng}):asc,price:desc"
}
```
- 🔴 Specifying parameters that do not conform to the `_geoPoint` signature causes the API to return an [invalid_search_parameter_sort](0061-error-format-and-definitions.md#invalid_search_parameter_sort) error.
- 🔴 Using `_geoDistance`, `_geo` or `_geoRadius` in a sort expression causes the API to return an[invalid_search_parameter_sort](0061-error-format-and-definitions.md#invalid_search_parameter_sort) error.

---

### **As an end-user, I want to know the document distance when I am sorting around a geo point.**

- Introduce a `_geoDistance` parameter to the search result `hit` object.

**`_geoDistance` field definition**

- Name: `_geoDistance`
- Description: Return document distance when the end-user sorts document from a `_geoPoint` in meters.
- Type: int
- Not required

> 💡 `_geoDistance` response field is only computed and shown when the end-user have sorted documents around a `_geoPoint`. So if the end-user filters documents using a `_geoRadius` built-in filter without sorting them around a `_geoPoint`, this field `_geoDistance` will not appear in the search response.

---

#### Related Ranking Rules Settings API Errors

- 🔴 Specifying a custom ranking rule with `_geo` or `_geoDistance` returns an [invalid_settings_ranking_rules](0061-error-format-and-definitions.md#invalid_settings_ranking_rules) error.
- 🔴 Specifying a custom ranking rule with `_geoPoint` returns an [invalid_settings_ranking_rules](0061-error-format-and-definitions.md#invalid_settings_ranking_rules) error.
- 🔴 Specifying a custom ranking rule with `_geoRadius` returns an [invalid_settings_ranking_rules](0061-error-format-and-definitions.md#invalid_settings_ranking_rules) error.

---

### IV. Finalized Key Changes

- Add a `_geo` reserved field on JSON and CSV format to index a geo point coordinates for a document.
- Add a `_geoPoint(lat, lng)` built-in sort rule.
- Add a `_geoRadius(lat, lng, distance_in_meters)` built-in filter rule.
- Return a `_geoDistance` in `hits` objects representing the distance in meters computed from the `_geoPoint` built-in sort rule.

## 2. Technical Aspects

### I. Measuring

- `filterableAttribute` setting definition to evaluate `_geo` presence.
- `sortableAttribute` setting definition to evaluate `_geo` presence.

## 3. Future Possibilities

- Add built-in filter to filter documents within `polygon` and `bounding-box`.
- Handling array of geo points in the document object.
- Handling multiple geo formats for the `_geo` field. e.g. "{lat},{lng}", a geohash etc.
- Handling distance in other formats (like the imperial format). **It's easy to implement on the user side though.**
- Handling position in other formats. It seems that [degrees and minutes](https://www.pacioos.hawaii.edu/voyager-news/lat-long-formats/) are also used a lot. **It's easy to implement on the user side though.**