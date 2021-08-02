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
- Filter documents by a given geo radius. It is possible to cumulate several geo-search filters within the `filter` field. `_geoRadius({lat}, {lng}, {distance_in_meters})`.
- Sort documents in ascending/descending order around a geo point. e.g. `_geoPoint({lat}, {lng}):asc`.
- It is possible to filter and/or sort by geographical criteria of the user's choice.
- There is no `geo` ranking rule that can be manipulated by the user. This one is integrated in the ranking rule `sort` by default.

### II. Motivation

According to our user feedback, the lack of a geo-search feature is mentioned as one of the biggest deal-breakers for choosing MeiliSearch as a search engine. A search engine must offer this feature. Some use cases specifically require integrated geo-search capabilities. Moreover, a lot of direct competitors offer it. Users today must find workarounds like using geohash to be able to geo-search documents. We hope to better serve the needs of users by implementing this feature. It allows multiplying the use-cases to which MeiliSearch can respond.

## 3. Future Possibilities

- Handling array of geo points in the document object.
- Handling multiple geo formats for the `_geo` field. e.g. "{lat},{lng}", a geo-hash etc..
- Filter documents in polygon and bounding-box.