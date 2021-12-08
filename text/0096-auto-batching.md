- Title: Auto-Batching
- Start Date: 2021-10-06
- Specification PR: [#96](https://github.com/meilisearch/specifications/pull/96)
- Discovery Issue: [#69](https://github.com/meilisearch/product/issues/69)

# Auto-Batching

## 1. Functional Specification

### 1. Summary

MeiliSearch can automatically group consecutive asynchronous `documentAdditions` tasks into a single batch for an index.

#### Summary Key points

- A `batchUid` field is added on a fully qualified `task` API object.

### 1.2 Motivation

We regularly tell users to batch their documents to speed up the indexing speed. We decided to integrate a simple scheduler to automatically batch consecutive document additions for an index to make it transparent for the users and enhance their quality of life using MeiliSearch.

### 1.3 Explanations

#### 1.3.1 Overall

All `tasks` are linked to specific a `batchUid` field. This `batchUid` field permits to identify that several identical, consecutive tasks have been grouped in the same batch.

Only consecutive `documentsAddition` `tasks` for the same index can have a shared `batchUid` since the scheduler will be able to group them together. This means that all `tasks` concerning other operations will also be part of a `batchUid` having only one task. This could be easily extended in the future by enhancing the scheduler capabilities.

This is to avoid having multiple incompressible computation time by grouping them together to increase performance when applied to a maximum number of documents.

#### 1.3.2 Scheduling tasks to a batch

The scheduling program that group document additions tasks within a single batch will only be triggered when a `task' is already being processed.

In other words, consecutive `documentsAddition` tasks on the same index will only be grouped together to be processed within the same batch when a waiting time is already present because MeiliSearch is already processing a task.

The more similar consecutives tasks the user sends in a row, the more likely the batching mechanism will be triggered.

#### 1.3.3 Closing a batch and processing it

When a batch is opened and gathering tasks, it can be closed and sent to be processed according to several criteria.

- All other incoming tasks not of type `documentAdditions` or concerning a different index close the current batching process.
- The batch closes and is written if a previously running task ends.
- ...

#### 1.3.4 Tweaking the scheduler behaviors
tbd

#### 1.3.5 Impacts on `task` API object format

- The different tasks grouped in a batch will be processed within the same transaction. That is to say that if a task fails within a batch. All the tasks fail or succeed.
- A `batchUid` field is added on fully-qualified `task` API objects.
- Tasks within the same batch share the same values for the `startedAt`, `finishedAt`, `duration` fields and the same error object in case an error occurs during the batch processing.

## 2. Technical Aspects
tbd

## 3. Future Possibilities

- Extends it for all consecutive payload types.
- Add a filter capability by `batchUid` on the /tasks endpoints.
- Schedule non-consecutive tasks.