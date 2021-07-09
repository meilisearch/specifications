- Title: Cancel pending tasks
- Start Date:
- Specification PR: https://github.com/meilisearch/specifications/pull/13
- MeiliSearch Issue: 
 - https://github.com/meilisearch/MeiliSearch/issues/1104
- Parent Spec: https://github.com/meilisearch/specifications/pull/12


# Cancel pending tasks

## 1. Feature Description and Interaction

### Summary

Allow to cancel tasks in the task queue or to cancel the entire coming tasks in the queue. A canceled task will have the status `canceled`. Only pending tasks will be cancellable. 

### Motivation

The task queue is convenient because it allows us to serialize tasks (updates/config/dumps) and assure the [ACID property](https://en.wikipedia.org/wiki/ACID) of MeiliSearch. It also causes some problems. As the task queue is an append-only queue, it is impossible to remove a task from this queue. The tasks can be large and can take a long time to index. Sometimes, it would be convenient to cancel a message in this queue or all the next pending tasks on it. 

### Additional Materials

As explained in the [parent spec](https://github.com/meilisearch/specifications/pull/12), It's quite rare to have this kind of task queue. Elasticsearch or Algolia, who have this kind of task queue, don't give the cancel or clear their 'tasks' queues. 

### Explanation

Two new routes and a new status (`canceled`) must be added to make this feature possible. 

A canceled task will stay at its place in the queue. So the typical answer of the get all task status route will be: 

```json
[
 {
 "status": "processed",
 "taskId": 1,
 ...
 },
 {
 "status": "failed",
 "taskId": 2,
 ...
 },
 {
 "status": "processed",
 "taskId": 3,
 ...
 },
 {
 "status": "pending",
 "taskId": 4,
 ...
 },
 {
 "status": "canceled",
 "taskId": 5,
 ...
 },
 {
 "status": "pending",
 "taskId": 6,
 ...
 },
 ...
]
```

the task status document will look like:

```json
{
 "status": "canceled",
 "taskId": 12,
 "type": {
 "name": "DocumentsAddition",
 "number": 4
 },
 "duration": 0,
 "enqueuedAt": "2019-12-07T21:16:09.623944Z",
 "processedAt": null,
 "canceledAt": "2019-12-07T21:16:09.703509Z", // date of cancelation
}
```

Until now, we will consider that every task status documents returned will have all fields. Even empty ones. 

For example:
- A pending task will look like:
```json
{
 "status": "pending",
 "taskId": 12,
 "type": {
 ...
 },
 "duration": null,
 "enqueuedAt": "2019-12-07T21:16:09.623944Z",
 "processedAt": null,
 "canceledAt": null
}
```

- A canceled task will look like:
```json
{
 "status": "canceled",
 "taskId": 12,
 "type": {
 ...
 },
 "duration": null,
 "enqueuedAt": "2019-12-07T21:16:09.623944Z",
 "processedAt": null,
 "canceledAt": "2019-12-07T21:16:09.703509Z"
}
```

- A processed/failed task will look like:
```json
{
 "status": "processed",
 "taskId": 12,
 "type": {
 ...
 },
 "duration": 120,
 "enqueuedAt": "2019-12-07T21:16:09.623944Z",
 "processedAt": "2019-12-07T21:16:09.703509Z",
 "canceledAt": null
}
```


#### HTTP API

**Cancel a task:**

- Method: PUT
- Route: `/indexes/:index_uid/tasks/:task_id/cancel`
 - index_uid: The index uid
 - task_id: The task identifier
- Status code: 204
- Response: No Body

**Cancel every pending task:**

- Method: PUT
- Route: `/indexes/:index_uid/tasks/cancel`
 - index_uid: The index uid
- Status code: 204
- Response: No Body

#### Potential errors

- The given task id does not exist: 
 - Status code: `404`
 - Error code: `task_not_found`
 - Error description: `The requested task can't be retrieved. Either it doesn't exist, or the database was left in an inconsistent state.`

- The given task id is a processed id: 
 - Status code: `400`
 - Error code: `uncancellable_task`
 - Error description: `The requested task id exists but could not be canceled. A processing/processed/failed task cannot be canceled.`

- Error during the cancel process:
 - Status code: `500`
 - Error code: `task_cancelion_failed`
 - Error description: `An error occurred during the task cancellation process. The requested cancellation has not been taken into account.`


### Impact on Documentation

- Add on the guide the mention of the `canceled` status. 
- Change the guide where we explain that it's not possible to cancel a task. 
- Add the two methods described in the HTTP API part of the documentation.
- Add the errors in the errors guide.

## 2. Technical Specifications

### Architecture
### Implementation Details

⚠️ When a task is canceled, the content itself should be deleted. It will free more disk.

### Corner Cases

## 3. Future Possibilities
- Cancel/Abort an in-process task.
- Add the option to cancel items with a filter based on the `type`.
- Add an option to cancel a range of items based on the `taskID`. 
- Cancel multiple tasks with comma-separated values (e.g. `/indexes/movies/tasks/12,13,14,15`). 

