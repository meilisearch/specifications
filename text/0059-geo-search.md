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
- Sort documents in ascending order around a geo point. e.g. `_geoPoint({lat}, {lng}):asc`. Descending order will not be supported for this first iteration.
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
- Format: `{lat:float, lng:float}`
- Not required

> 💡 if `_geo` is found in the document payload, `lat` and `lng` are required.
> 💡 `lat` and `lng` must be of float value.

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
            "lat": 48.862725,
            "lng": 2.287592
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

- 🔴 Giving a bad formed `_geo` that do not conform to the format causes the `update` payload to fail. A new `invalid_geo_field` error is given in the `update` object.

##### Errors Definition

## invalid_geo_field

### Context

This error occurs when the `_geo` field of a document payload is not valid.

### Error Definition

```json
{
    "message": "The _geo field is invalid. :syntaxErrorHelper.",
    "code": "invalid_geo_field",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_geo_field"
}
```

- The `:syntaxErrorHelper` is inferred when a syntax error is encountered.

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

- 🔴 Specifying parameters that do not conform to the `_geoRadius` signature causes the API to return an `invalid_filter` error. The error message should indicate how `_geoRadius` should be used. See `_geoRadius` built-in filter rule definition part.
- 🔴 Using `_geo`, `_geoDistance`, `_geoPoint` in a filter expression cause the API to return an `invalid_filter` error. `message` should be `:reservedKeyword is a reserved keyword and thus can't be used as a filter expression.`

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
>`_geoPoint` built-in sort rule can sort documents in ascending order only.
>
> The `:desc` order is not supported due to a technical limit. See Technical Aspects part for more details.

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
- 🔴 Specifying parameters that do not conform to the `_geoPoint` signature causes the API to return an `invalid_sort` error. The error message should indicate how `_geoPoint` should be used. See `_geoPoint` built-in sort rule definition part.
- 🔴 Specifying `:desc` for a `_geoPoint` sort rule will raise an `invalid_sort` error with a message explaining that `_geoPoint` can only be used with `:asc` order.
- 🔴 Using `_geo`, `_geoDistance`, `_geoRadius` in a sort expression cause the API to return an `invalid_sort` error. `message` should be `:reservedKeyword is a reserved keyword and thus can't be used as a sort expression.`

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

### `invalid_criterion` error changes

The error is currently marked as an internal error thus the name is not explicit and consistent with the term `Ranking Rule` a user can encounter in the documentation and in the API resource name. A new definition of this error is proposed.

#### invalid_ranking_rule

#### Context

This error is raised asynchronously when the user try to specify an invalid ranking rule in the ranking rules setting.

#### Error Definition

```json
    "message": ":rankingRule ranking rule is invalid. Valid ranking rules are Words, Typo, Sort, Proximity, Attribute, Exactness and custom ranking rules."
    "code": "invalid_ranking_rule"
    "type": "invalid_request"
    "link": "https://docs.meilisearch.com/errors#invalid_ranking_rule"
```

- 🔴 Specifying an invalid ranking rule name raises an `invalid_ranking_rule` error. See `message` defined in the error definition part.
- 🔴 Specifying a custom ranking rule with `_geo` or `_geoDistance` raises an `invalid_ranking_rule` error. The message is `:reservedKeyword is a reserved keyword and thus can't be used as a ranking rule.`.

---

### IV. Finalized Key Changes

- Add a `_geo` reserved field on JSON and CSV format to index a geo point coordinates for a document.
- Add a `_geoPoint(lat, lng)` built-in sort rule.
- Add a `_geoRadius(lat, lng, distance_in_meters)` built-in filter rule.
- Return a `_geoDistance` in `hits` objects representing the distance in meters computed from the `_geoPoint` built-in sort rule.

## 2. Technical Aspects

### I. :desc case - Sorting documents around a geo point

We may encounter technical difficulties to implement a descending order capability for the geo sorting. This first iteration will allow us to identify if this is a real technical problem. If we verify the existence of this problem, we will think at this moment of the best solution to bring on the table.

> 💡 In a first step, we could not allow `:desc` on a geoPoint if it's a complex technical issue.

---

Edit-date: 2021-08-31

As imagined during the first phases of discovery we encounter a technical difficulty to compute a descending order around a `_geoPoint`.

The technical team will have to do some preparatory work to overcome this limitation. For the time being, it is not planned to spend more time on this as it is not a primary use case. The impact may be very small, so it's not worth the effort at this point.

### Decision taken

It was decided after a discussion with @irevoire and @Kerollmops that when the user specifies the sort rule `_geoPoint(lat, lng):desc` an error `invalid_sort` will be returned with a message explaining that `:desc` is not available.

#### Why keep the :asc if :desc is not valid?

To keep consistency and not to introduce a different syntax among the `sort` search parameter. This is something we'll re-evaluate later.

---

### II. Measuring

- `filterableAttribute` setting definition to evaluate `_geo` presence.
- `sortableAttribute` setting definition to evaluate `_geo` presence.

## 3. Future Possibilities

- Add built-in filter to filter documents within `polygon` and `bounding-box`.
- Handling `:desc` order around a geoPoint
- Handling array of geo points in the document object.
- Handling multiple geo formats for the `_geo` field. e.g. "{lat},{lng}", a geohash etc.
