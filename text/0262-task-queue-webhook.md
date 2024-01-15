# Task queue webhook

## 1. Summary

Describe the usage of a webhook URL that will be called whenever a task finishes so a third party can be notified.
It is used to compute some of the required metrics for the Monitoring Metrics v1 project. Heavily related to the
format of the [`tasks`](/text/0060-tasks-api.md#fully-qualified-task-object).

## 2. Motivation

It'll give users the possibility to wait on task completion _without_ having to poll meilisearch over and over again.
This heavily reduces the network congestion and will allow us to compute the different metrics/ charts needed for the
[Monitoring Metrics v1](https://www.notion.so/ Monitoring-Metrics-v1-4782d56795c043799dde33309e73a20f? pvs=21) project
that is related to the tasks queue (*Indexing latency, tasks operations*), we need to receive task information on the
cloud stack.

## 3. Functional Specification

### 3.1. Summary Key Points

- The webhook URL is configured either via the env var `MEILI_TASK_WEBHOOK_URL` or via the CLI option flag `--task-webhook-url`
    - By default, the webhook URL value is empty.
    - If the given value does not match a valid URL format, a human-readable error is returned to the user, and the engine does not start.
    - A valid user-provided URL is kept as is.
    - You can optionally set an Authorization header by using the `--task-webhook-authorization-header` CLI option flag or via the `MEILI_TASK_WEBHOOK_AUTHORIZATION_HEADER` env var.
- When a valid webhook URL is configured and when a batch of tasks reaches a finished status (`status: succeeded/failed`), the engine sends a `POST` request to the user-configured webhook URL. The HTTP request contains the finished tasks in its payload.
    - The sent payload is compressed with gzip and represented in the JSON Lines text format.
- A task sent over the webhook URL matches the [task object definition](https://github.com/meilisearch/specifications/blob/main/text/0060-tasks-api.md#1-task-object-definition).
- If an error is encountered while sending the payload to the URL, Meilisearch logs it but does not stop processing tasks.
- Payload example
    - In that case, the user configured `https://myproject.com/mywebhook?common=people`
```json
//POST HTTP request to https://myproject.com/mywebhook?common=people

{"uid":4,"indexUid":"movie","status":"failed","type":"indexDeletion","canceledBy":null,"details.deletedDocuments":0,"error.message":"Index `movie` not found.","error.code":"index_not_found","error.type":"invalid_request","error.link":"https://docs.meilisearch.com/errors#index_not_found","duration":"PT0.001192S","enqueuedAt":"2022-08-04T12:28:15.159167Z","startedAt":"2022-08-04T12:28:15.161996Z","finishedAt":"2022-08-04T12:28:15.163188Z"}
{"uid":5,"indexUid":"movie","status":"failed","type":"indexDeletion","canceledBy":null,"details.deletedDocuments":0,"error.message":"Index `movie` not found.","error.code":"index_not_found","error.type":"invalid_request","error.link":"https://docs.meilisearch.com/errors#index_not_found","duration":"PT0.001192S","enqueuedAt":"2022-08-04T12:28:15.159167Z","startedAt":"2022-08-04T12:28:15.161996Z","finishedAt":"2022-08-04T12:28:15.163188Z"}
{"uid":6,"indexUid":"movie","status":"failed","type":"indexDeletion","canceledBy":null,"details.deletedDocuments":0,"error.message":"Index `movie` not found.","error.code":"index_not_found","error.type":"invalid_request","error.link":"https://docs.meilisearch.com/errors#index_not_found","duration":"PT0.001192S","enqueuedAt":"2022-08-04T12:28:15.159167Z","startedAt":"2022-08-04T12:28:15.161996Z","finishedAt":"2022-08-04T12:28:15.163188Z"}
```

---

### 3.1. CLI Definition

You can find the CLI information about the webhook [here](/text/0119-instance-options.md)

## 4. Technical Aspects

### 4.1. Sending a batch

While a batch is being sent, the scheduler won't process a new batch until the webhook accepts the payload.

## 5. Future Possibilities

- Let users subscribe to a webhook to specific indexes or tasks instead of providing one global hook for everyone.
- Let users update the URL of their webhook at runtime.
- Stops blocking the processing of new tasks while we're sending a payload.
