- Title: Words position limit
- Start Date: 2021-10-06
- Specification PR: [#77](https://github.com/meilisearch/specifications/pull/77)
- Discovery Issue: [#202](https://github.com/meilisearch/product/issues/202)

# Words position limit

## 1. Functional Specification

### I. Summary

The purpose of this specification is to remove the limit of 1000 positions per attribute.

#### Summary Key points

- 1000 positions limit per document field is now raised at 65535.

### II. Motivation

We've seen many users denormalizing fields into multiple fields to index all the words because of the initial limit. This change will increases the limit to 65535, which should greatly reduce frictions on this issue. We expect to reduce the changes to be made to the document schema in order to use MeiliSearch more quickly and easily.

### III. Technical Explanations
n/a

## 2. Technical Aspects

When MeiliSearch indexes a document, it indexes several word positions per field until a limit is reached.

It is important to note that the limit is not strictly related to the number of words. Indeed, soft separators are also counted as `1` position while hard separators are counted as `8` positions.

## 3. Future Possibilities

- Expose a configurable default limit up to 65535.