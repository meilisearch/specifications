* Title: Asynchronous Updates
* Start Date:
* Specification PR: https://github.com/meilisearch/specifications/pull/12
* MeiliSearch Issue:

# Asynchronous updates

## 1\. Feature Description and Interaction

### Summary

This spec is the spec of a feature already in place. 

It's scope is:
- The name for update list and update identifiers. 
- The route to get one update with this identifier.
- The route to get all updates. 
- The update's format (JSON).

### Motivation

MeiliSearch is an asynchronous API. It means that the API does not behave as you would expect when handling the request's responses. Some operations are put in a queue and will be executed in turn (asynchronously).

We must define how to interact with this update queue.

### Additional Materials

In Elasticsearch, the updates are not put in an update queue. They put the HTTP client on hold for the indexing process time. This is the case for all types of updates ([Update](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-update.html), [Bulk](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-bulk.html), [Update By Query](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-update-by-query.html), or [Reindex](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-reindex.html)). Only for most of these methods, it is possible to pass an argument (`wait_for_completion=false`) to make updates in [tasks](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/tasks.html). Their task system is similar to our update system. But, it is much more agnostic, and it is possible to query the past tasks accurately. 

On the Algolia side, they also call the update queue [tasks](https://www.algolia.com/doc/rest-api/search/#get-a-tasks-status). So, they return a `taskID` rather than `updateID` for each request for document or settings updates. 

Other databases don't seem to have an async update queue. 

**TL;DR:**
- The other solutions call `updates`, `tasks`.
- Most other solutions allow sending synchronous updates. 

### Explanation

#### Async flow

* When making a written request (create/update/delete) against the search engine, it stores the operation received in a queue and returns a updateId. With this id, the operation update is trackable.
* Each update received is treated following the order it has been received.
* You can get the update status on the /updates route.
* Processed updates are marked as processed and kept in the operation list (available at `/indexes/:index\_uid/updates`). They won't be deleted.

#### Which operations are async?

Every operation which could be compute-expensive is asynchronous. 

These include:

* Update index settings
* Add/update/delete documents

These not include:

* Create/update/delete an index
* Create a dump

#### HTTP API

**Get an update status:** 

- Method: GET 
- Route: `/indexes/:index_uid/updates/:update_id` 
     - index_uid: The index unique identifier
     - update_id: The update identifier
- Response: An update object (see examples)
- Status Code: 200 Ok

**Get all update status:**

- Method: GET 
- Route: `/indexes/:index_uid/updates` 
     - index_uid: The index unique identifier
- Response: An array of update object (see examples)
- Status Code: 200 Ok


#### Understanding updates

Updates return the following information:

* **status**: The status of the operation (enqueued, processed, or failed).
* **updateId**: The identifier of the update.
* **type**: The type of the operation.
* **enqueuedAt:** The date at which the operation has been added to the queue.
* **processedAt**: The date at which the operation has been processed.

#### Examples

- Adding documents:

```json
{
  "status": "processed",
  "updateId": 1,
  "type": {
    "name": "DocumentsAddition",
    "number": 19653
  },
  "duration": 12.757581815,
  "enqueuedAt": "2019-12-07T21:10:07.607581330Z",
  "processedAt": "2019-12-07T21:10:20.511525620Z"
}
```

- Failing to upload a document:

```json
{
  "status": "failed",
  "updateId": 3,
  "type": {
    "name": "DocumentsAddition",
    "number": 1
  },
  "error": "document id is missing",
  "duration": 0.000048524,
  "enqueuedAt": "2019-12-07T20:23:50.156433207Z",
  "processedAt": "2019-12-07T20:23:50.157436246Z"
}
```

### Impact on Documentation

* We already documented the update queue in a guide: https://docs.meilisearch.com/guides/advanced\_guides/asynchronous\_updates.html#asynchronous-updates
* The API to get one or several updates status is already written: https://docs.meilisearch.com/references/updates.html

## 2\. Technical Specifications

### Architecture

Already Implemented

### Implementation Details

Already Implemented

### Corner Cases

N.A.

## 3\. Future Possibilities

- Add the possibility to clear the update queue.
- Add the option to remove one enqueued element.
- Compact old elements.
- Get all updates with some filters.
- Improve the error system. 
