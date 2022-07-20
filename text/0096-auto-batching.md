- Title: Auto-Batching

# Auto-Batching

## 1. Summary

Meilisearch can automatically group consecutive asynchronous `documentAddition` or `documentPartial` tasks for the same index via an automatic batching mechanism.

The user can enable this auto-batching behavior through various command flag options.

## 2. Motivation

We have regularly collected user pain points pointing out the slow indexing over the last year. We explained several times to users to make batches containing a maximum of documents to be updated/added to compress the indexing time of specific data structures.

To make Meilisearch easier to use, we explored the idea of automatically creating these batches within Meilisearch before indexing users’ documents.

## 3. Functional Specification

### 3.1. Explanations

All `tasks` are part of a batch identified by a `batchUid`. A task batch preserves the logical order of the tasks for a given index.

Only consecutive `documentAdditionOrUpdate` tasks for the same index can have the same `batchUid`. All `tasks` concerning other operations will also be part of a `batchUid` having only one task.

#### 3.1.1. Grouping tasks to a single batch

The scheduling program that groups tasks within a single batch is triggered when an asynchronous `task` currently processed reaches a terminal state as `succeeded` or `failed`.

In other words, when a scheduled `documentAdditionOrUpdate` task for a given index is picked from the task queue, the scheduler fetches and groups all `documentAdditionOrUpdate` tasks for that same index in a batch.

The more similar consecutive tasks the user sends in a row, the more likely the batching mechanism can group these tasks.

##### 3.1.1.1. Schema

![Auto-batching Process](https://user-images.githubusercontent.com/3692335/145787054-4cb07b5e-c80e-498a-8843-d0cc46329e9b.png)

##### 3.1.1.2. `batchUid` generation

The batch identifiers are unique and strictly increasing.

#### 3.1.2. Impacts on `task` API resource

- The different tasks grouped in a batch are processed within the same transaction. But if a task fails within a batch, the whole batch does not fail, only the related task.
- A `batchUid` field is added on fully-qualified `task` API objects. `batchUid` values are unique and strictly increasing.
- Tasks within the same batch share the same values for the `startedAt`, `finishedAt`, `duration` fields, and the same `error` object if an error occurs for a `task` during the batch processing.
- If a batch contains many `tasks`, the `task` `details` `indexedDocuments` is identical in all `tasks` belonging to the same processed `batch`.

### 3.2. Auto-batching mechanisms options

### 3.2.1. `--disable-auto-batching`

By default, the auto-batching feature is enabled.

The auto-batching feature can be desactivated by passing the command flag `--disable-auto-batching` (or the environment variable `MEILI_DISABLE_AUTO_BATCHING`) to Meilisearch at launch.

## 4. Technical Aspects
N/A

## 5. Future Possibilities

- Extends it for all consecutive payload types.
- Add a filter capability by `batchUid` on the `/tasks` endpoints.
- <s>Do not fail the entire transaction if a document is not valid. Report the documents that could not be indexed to the user.</s>
- <s>Enable auto-batching by default.</s>
- Optimize some tasks sequence, for example if there is a document addition followed by an index deletion, we could skip the document addition
