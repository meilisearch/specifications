- Title: Geo-search
- Start Date: 2021-08-02
- Specification PR: [#59](https://github.com/meilisearch/specifications/pull/59)
- MeiliSearch Tracking-issues:

# Geo-search

## 1. Functional Specification

### I. Summary

The purpose of this specification is to add a first iteration of the **geo-search** feature to give geo-filtering and geo-sorting capabilities at search time.

#### Summary Key points

- Documents MUST have a `_geo` reserved object to be geo-searchable.
- Filter documents by a given geo radius using the built-in filter `_geoRadius({lat}, {lng}, {distance_in_meters})`. It is possible to cumulate several geo-search filters within the `filter` field. .
- Sort documents in ascending/descending order around a geo point. e.g. `_geoPoint({lat}, {lng}):asc`.
- It is possible to filter and/or sort by geographical criteria of the user's choice.
- There is no `geo` ranking rule that can be manipulated by the user. This one is automatically integrated in the ranking rule `sort` by default and activated by sorting using the `_geoPoint({lat}, {lng})` built-in sort rule.
- Using `_geoPoint({lat}, {lng})` in the `sort` parameter at search leads the engine to return a `_geoDistance` within the search results. This field represents the distance in meters of the document from the specified `_geoPoint`.

### II. Motivation

According to our user feedback, the lack of a geo-search feature is mentioned as one of the biggest deal-breakers for choosing MeiliSearch as a search engine. A search engine must offer this feature. Some use cases specifically require integrated geo-search capabilities. Moreover, a lot of direct competitors offer it. Users today must find workarounds like using geohash to be able to geo-search documents. We hope to better serve the needs of users by implementing this feature. It allows multiplying the use-cases to which MeiliSearch can respond.

### III. Technical Explanations

#### **As a developer, I want to add geo-spatial coordinates to a document so that the document can be geo-searchable.**

- Introduce a reserved field `_geo` for documents to store geo spatial data from an **object** made of `lat` and `lng` fields for a **JSON format**.
- Introduce a reserved column `_geo` for documents to store geo spatial data from a **string** made of `lat,lng` for a **CSV format**.

##### **JSON Format**

**`_geo` field definition**

- Name: `_geo`
- Type: Object
- Format: `{lat:float, lng:float}`
- Not required

💡 if `_geo` is found in the document payload, `lat` and `lng` are required.
💡 `lat` and `lng` must be of float value.

##### **CSV Format**

Following the format already defined in the https://github.com/meilisearch/specifications/pull/28/files specification for document indexing from a CSV format. A reserved column `_geo` can be added to specify the geographical coordinates of a document.

e.g.
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
```
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

```
{
    "updateId": 1
}
```

#### PUT Add or replace documents `/indexes/{indexUid}/documents`

##### Request body
```
[
    {
        "id": 1,
        "brand": "F40 LM",
        "brand": "Ferrari",
        "_geo": {
            "lat": 48.862725,
            "lng": 2.287592
        }
    }
]
```

##### 202 Accepted - Response body

```
{
    "updateId": 2
}
```

### **As a end-user, I want to filter documents by a geo radius.**

- Introduce a `_geoRadius({lat}, {lng}, {distance_in_meters})` built-in filter rule  to `filter` documents in a geo radius.shape.

**`_geoRadius` built-in filter rule definition**

- Name: `_geoRadius`
- Signature: ({lat:float}:required, {lng:float}:required, {distance_in_meters:int}:required)
- Not required

#### GET Search `/indexes/{indexUid}/search`

```
...&filter="brand=Mercedes AND _geoRadius(48.862725, 2.287592, 2000)"`
```

#### POST Search `/indexes/{indexUid}/search`

```
{
    "filter": ["brand = Ferrari", "_geoRadius(48.862725, 2.287592, 2000)"]
}
```

> 🔴 Specifying parameters that do not conform to the `_geoRadius` signature causes the API to return an `invalid_filter` error. The error message should indicate how `_geoRadius` should be used. See `_geoRadius` built-in filter rule definition part.

### **As a end-user, I want to sort documents around a geo point.**

- Introduce a `_geoPoint({lat}, {lng})` function parameter to `sort` documents around a central point.

**`_geoPoint` built-in sort rule definition**

- Name: `_geoPoint`
- Signature: ({lat:float}:required, {lng:float}:required)
- Not required

Following the [`sort` specification feature](https://github.com/meilisearch/specifications/pull/55):
- The `_geo` field does not need to be referred to as `sortableAttributes` by the developer.
- There is no `geo` ranking rule as such. It is in fact within the `sort` ranking rule in an obfuscated way.
- `_geoRadius` built-in sort rule can sort documents in ascending or descending order.

#### GET Search `/indexes/{indexUid}/search`

```
    ...&sort=_geoPoint({lat, lng}):asc,price:desc
```

#### POST Search `/indexes/{indexUid}/search`

```
{
    "sort": "_geoPoint({lat, lng}):asc,price:desc"
}
```
> 🔴 Specifying parameters that do not conform to the `_geoPoint` signature causes the API to return an `invalid_sort` error. The error message should indicate how `_geoPoint` should be used. See `_geoRadius` built-in sort rule definition part.

### **As a end-user, I want to know the document distance when I am sorting around a geo point.**

- Introduce a `_geoDistance` parameter to the search result `hit` object.

**`_geoDistance` field definition**

- Name: `_geoDistance`
- Description: Return document distance from a `_geoPoint` in meters.
- Type: int
- Not required

> 💡 `_geoDistance` response field is only computed and shown when the end-user have sorted documents around a `_geoPoint`. Filtering documents by a `_geoRadius` does not calculate and display the `_geoDistance` field within the search results.


### IV. Finalized Key Changes

## 2. Technical Aspects

## 3. Future Possibilities

- Handling array of geo points in the document object.
- Handling multiple geo formats for the `_geo` field. e.g. "{lat},{lng}", a geo-hash etc..
- Filter documents in polygon and bounding-box.