# Geosearch

## 1. Summary

Meilisearch provides geosearch features. It's possible to filter and sort documents based on their geo-spatial coordinates.

## 2. Motivation

Some use cases specifically require integrated geosearch capabilities.

## 3. Functional Specification

### 3.1. Define Geo-spatial Coordinates

The reserved field `_geo` define coordinates within a document.

#### 3.1.1. JSON `_geo` Field Definition

- Name: `_geo`
- Type: Object
- Format: `{lat:number|string, lng:number|string}`
- Not required

> 💡 if `_geo` is found in the document payload, `lat` and `lng` are required.
> 💡 `lat` and `lng` field type can be mixed. e.g. `lat` can be a string while `lng` is a number in the same `_geo` object.

#### 3.1.2. CSV `_geo` Column Definition

A reserved column `_geo` can be added to specify the geographical coordinates of a document within a csv file.

csv format example
```
"id:number","label","brand","_geo"
"1","F40","Ferrari","48.862725,2.287592"
```

- Name: `_geo`
- Type: String
- Format: `"lat:float,lng:float"`
- Not required

### 3.2. Enable Geo-search Features

- To filter documents based on their geo-spatial coordinates, the `_geo` field must be defined in the [`filterableAttributes`](0123-filterable-attributes-setting-api.md) index setting.
- To sort documents based on their geo-spatial coordinates, the `_geo` field must be defined in the [`sortableAttributes`](0123-sortable-attributes-setting-api.md) index setting.

#### 3.2.1. Asynchronous Error

As soon as `_geo` is specified in one of these settings, an asynchronous error may occur when indexing a `_geo` field.

- 🔴 Giving a bad formed `_geo` that do not conform to the format returns an [`invalid_geo_field`](0061-error-format-and-definitions#invalid_geo_field) error.

### 3.3. Filtering Documents

>  The `_geo` field has to be set in `filterableAttributes` index setting by the developer to activate geo filtering capabilities at search.

The following built-in filter rules can be used as expressions within the `filter` parameter for a search query.

#### 3.3.1. Supported Shapes

##### 3.3.1.1. Radius

`_geoRadius({lat}, {lng}, {distance_in_meters})` built-in filter rule can be used to filter documents in a geo radius shape.

- Name: `_geoRadius`
- Signature: ({lat:float}:required, {lng:float}:required, {distance_in_meters:int}:required)
- Not required
- `distance_in_meters` only accepts positive value.

##### 3.3.1.2. BoundingBox

`_geoBoundingBox(({lat}, {lng}), ({lat, lng}))` built-in filter can be used to filter documents in a geo bounding box shape.

- Name: `_geoBoundingBox`
- Signature: (({lat:float}:required, {lng:float}:required), ({lat:float}:required, {lng:float}:required))
- Not required


#### 3.3.2. Errors

- 🔴 Specifying parameters that do not conform to the `_geoRadius` signature returns an [`invalid_filter`](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Specifying parameters that do not conform to the `_geoBoundingBox` signature returns an [`invalid_filter`](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Using `_geoDistance`, `_geo`, or `_geoPoint` in a filter expression returns an [`invalid_filter`](0061-error-format-and-definitions.md#invalid_filter) error.

### 3.4. Sorting Documents

>  The `_geo` field has to be set in `sortableAttributes` index setting by the developer to activate geo sorting capabilities at search.

The following built-in filter rules can be used as expressions within the `sort` parameter for a search query.

#### 3.4.1. Sorting Around A Geo Point

`_geoPoint({lat}, {lng}):asc|desc` built-in sort can be used to sort documents around a geo-spatial point.

- Name: `_geoPoint`
- Signature: ({lat:float}:required, {lng:float}:required)
- Not required

>There is no `geo` ranking rule as such.
>
>`_geoPoint` built-in sort rule can sort documents in ascending/descending order e.g. `_geoPoint({lat, lng}):asc` / `_geoPoint({lat, lng}):desc`

##### 3.4.1.1. `_geoDistance` Search Result Field

When the sort expression `_geoPoint` is used, Meilisearch returns the distance in meters relative to the coordinates of the `_geoPoint` for each search result in a `_geoDistance` field.

- Name: `_geoDistance`
- Description: Return the document distance, in meters, from the specified `_geoPoint` coordinates.
- Type: Integer
- Not required

> 💡 `_geoDistance` response field is only computed and shown when the end-user have sorted documents around a `_geoPoint`. So if the end-user filters documents using a `_geoRadius`, or a `_geoBoundingBox` filter expression without sorting them around a `_geoPoint`, this field `_geoDistance` will not appear in the search response.

##### 3.4.1.2. Errors

- 🔴 Specifying parameters that do not conform to the `_geoPoint` signature returns an [`invalid_sort`](0061-error-format-and-definitions.md#invalid_sort) error.
- 🔴 Using `_geoDistance`, `_geo`, `_geoRadius`, or `_geoBoundingBox` in a sort expression returns an [`invalid_sort`](0061-error-format-and-definitions.md#invalid_sort) error.

---

## 4. Technical Details
N/A

## 5. Future Possibilities

- Support `_geoPolygon` filter
- Handling array of geo points in the document object.
- Handling multiple geo formats for the `_geo` field. e.g. "{lat},{lng}", a geohash etc.
- Handling distance in other formats (like the imperial format). **It's easy to implement on the user side though.**
- Handling position in other formats. It seems that [degrees and minutes](https://www.pacioos.hawaii.edu/voyager-news/lat-long-formats/) are also used a lot. **It's easy to implement on the user side though.**