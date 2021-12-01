- Title: Auto-Batching
- Start Date: 2021-10-06
- Specification PR: [#96](https://github.com/meilisearch/specifications/pull/96)
- Discovery Issue: [#69](https://github.com/meilisearch/product/issues/69)

# Auto-Batching

## 1. Functional Specification

### 1. Summary

#### Summary Key points

### 1.2 Motivation

### 1.3 Explanations

## 2. Technical Aspects

## 3. Future Possibilities

- Extends it for all payload type.
- Add a filter capability by `batchId` on the /tasks endpoints.
- Add a `batch` object to the `task` object to isolate `duration`, `batchId` and `processedAt` at batch level.
- Reorder non-consecutive tasks with a scheduler mechanism.