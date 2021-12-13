- Title: Auto-Batching
- Start Date: 2021-10-06
- Specification PR: [#96](https://github.com/meilisearch/specifications/pull/96)
- Discovery Issue: [#69](https://github.com/meilisearch/product/issues/69)

# Auto-Batching

## 1. Functional Specification

### 1.1 Summary

MeiliSearch can automatically group consecutive asynchronous `documentsAddition` or `documentsPartial` tasks for the same index into a single batch via an automatic batching mechanism.

#### Summary Key points

- A `batchUid` field is added on a fully qualified `task` API object.
- An auto-batching mechanism groups consecutive `documentsAddition` or consecutive `documentsPartial` enqueued tasks for a similar index when a task is fetched from the FIFO task queue to be processed.

### 1.2 Motivation

We regularly tell users to batch their documents to speed up the indexing speed. We decided to integrate a simple scheduler to automatically batch consecutive tasks for an index to make it transparent for the users and enhance their quality of life using MeiliSearch.

### 1.3 Explanations

#### 1.3.1 Overall

All `tasks` are linked to a specific `batchUid` field. This `batchUid` field permits to identify that several identical, consecutive tasks have been grouped in the same batch.

Only consecutive `documentsAddition` and `documentsPartial` `tasks` for the same index can have a shared `batchUid` since the scheduler will be able to group them together. This means that all `tasks` concerning other operations will also be part of a `batchUid` having only one task. This could be extended in the future by enhancing the scheduler capabilities.

This is to avoid having multiple incompressible computation time by grouping them together to increase performance when applied to a maximum number of documents.

#### 1.3.2 Grouping tasks to a single batch

The scheduling program that groups tasks within a single batch is triggered when an asynchronous `task` currently processed reaches a terminal state as `succeeded` or `failed`.

In other words, when the next `task` should be picked from the FIFO task queue. The scheduler fetch and group all consecutive `documentsAddition` for a similar index in a batch until it encounters another task type or a similar task type but for a different index.

> Note that we are considering implementing a configurable limit of maximum documents to process for a batch and a flag to activate the auto-batching mechanism.

The more similar consecutive tasks the user sends in a row, the more likely the batching mechanism will be able to group these tasks in a batch. It can be seen as an automatic back-pressure mechanism.

##### 1.3.2.1 Schema

![Auto-batching Process](https://user-images.githubusercontent.com/3692335/145787054-4cb07b5e-c80e-498a-8843-d0cc46329e9b.png)

##### 1.3.2.2 `batchUid` generation

The identifier chosen for the `batchUid`field corresponds to the `uid` value of the first task grouped within a batch. The batch identifiers are therefore unique and consecutive.

#### 1.3.3 Impacts on `task` API object format

- The different tasks grouped in a batch will be processed within the same transaction. That is to say that if a task fails within a batch, all the tasks fail or succeed.
- A `batchUid` field is added on fully-qualified `task` API objects.
- Tasks within the same batch share the same values for the `startedAt`, `finishedAt`, `duration` fields, and the same `error` object if an error occurs for a `task` during the batch processing.

#### 1.3.4 Tweaking the auto-batching mechanism
tbd

## 2. Technical Aspects
tbd

## 3. Future Possibilities

- Extends it for all consecutive payload types.
- Add a filter capability by `batchUid` on the /tasks endpoints.
- Schedule non-consecutive tasks.