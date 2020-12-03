- Title: Abort enqueued updates
- Start Date:
- Specification PR: https://github.com/meilisearch/specifications/pull/13
- MeiliSearch Issue: 
    - https://github.com/meilisearch/MeiliSearch/issues/1104
- Parent Spec: https://github.com/meilisearch/specifications/pull/12


# Abort enqueued updates

## 1. Feature Description and Interaction

### Summary

Allow to abort messages in the update queue or to empty the entire queue. An aborted update will have the status `aborted`.

Questions not answered:
- We have an attribute on update info `processedAt`. What becomes this attribute if we add the possibility to abort an update? We already use this term in the case of a failed update.

### Motivation

The update queue is convenient because it allows us to serialize updates and assure the [ACID property](https://en.wikipedia.org/wiki/ACID) of MeiliSearch. It also causes some problems. As the queue message is an append-only queue, it is impossible to remove an update from this queue. The updates can be large and can take a long time to index. Sometimes, it would be convenient to abort a message in this queue or empty it. 

### Additional Materials

As explained in the [parent spec](https://github.com/meilisearch/specifications/pull/12), It's quite rare to have this kind of update queue. Elasticsearch or Algolia, who have this kind of update queue, don't give the abort or clear their 'tasks' queues. 

### Explanation

Two new routes and a new status (`aborted`) must be added to make this feature possible. 

An aborted update will stay at its place in the queue. So the typical answer of the get all update status route will be: 

```json
[
    {
        "status": "processed",
        "updateId": 1,
        ...
    },
    {
        "status": "failed",
        "updateId": 2,
        ...
    },
    {
        "status": "processed",
        "updateId": 3,
        ...
    },
    {
        "status": "enqueued",
        "updateId": 4,
        ...
    },
    {
        "status": "aborted",
        "updateId": 5,
        ...
    },
    {
        "status": "enqueued",
        "updateId": 6,
        ...
    },
    ...
]
```

the update status document will look like:

```json
{
  "status": "aborted",
  "updateId": 12,
  "type": {
    "name": "DocumentsAddition",
    "number": 4
  },
  "duration": 0,
  "enqueuedAt": "2019-12-07T21:16:09.623944Z",
  "processedAt": "2019-12-07T21:16:09.703509Z" // date of abortion
}
```

#### HTTP API

**Abort an update:**

- Method: DELETE
- Route: `/indexes/:index_uid/updates/:update_id`
    - index_uid: The index uid
    - update_id: The update identifier
- Status code: 204
- Response: No Body

** abort all update:**

- Method: DELETE
- Route: `/indexes/:index_uid/updates`
    - index_uid: The index uid
- Status code: 204
- Response: No Response

#### Potential errors

- The given update id does not exist: 
    - Status code: `404`
    - Error code: `update_not_found`
    - Error description: `The requested update can't be retrieved. Either it doesn't exist, or the database was left in an inconsistent state.`

- The given update id is a processed id: 
    - Status code: `400`
    - Error code: `unabortable_update`
    - Error description: `The requested update id exists but could not be aborted. A processing/processed/failed update cannot be aborted.`

- Error during the abort process:
    - Status code: `400`
    - Error code: `update_abortion_failed`
    - Error description: `An error occurred during the update abortion process. The requested abortion has not been taken into account.`


### Impact on Documentation

- Add on the guide the mention of the `aborted` status. 
- Change on the guide where we explain that it's not possible to abort an update. 
- Add the two methods described in the HTTP API part of the documentation.

## 2. Technical Specifications

### Architecture
### Implementation Details

⚠️ When an update is aborted, the content itself should be deleted. It will free more disk.

### Corner Cases

## 3. Future Possibilities
- Abort an in-process update.
- Add the option to abort items with a filter based on the `type`.
- Add an option to abort a range of items based on the `updateID`. 
- Abort multiple updates with comma-separated values (e.g. `/indexes/movies/updates/12,13,14,15`). 
