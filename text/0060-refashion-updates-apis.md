- Title: Refashion Updates APIs
- Start Date: 2021-08-13
- Specification PR: [#55](https://github.com/meilisearch/specifications/pull/60)
- Discovery Issue: [#43](https://github.com/meilisearch/product/issues/43)

# Refashion Updates APIs

## 1. Functional Specification

### I. Summary

The term `update` is not the best choice as it can be confused with document updates or `settings`. We have chosen to replace this term with `task`, which is more generic and better reflects the meaning of this API resource.

As an additional change, we have reworked the format of an update to make it more in line with our expectations of an API that is supposed to be easily understandable and developer-oriented.

The changes we will make to the response format of the `task` object lists will make it easier to add more functionality in a future iteration. See the Future Possibilities section for a brief overview.

Two new API endpoints are added. Although quite simple, they allow to consult the list of tasks or a specific task without being forced to know the related index.

#### Summary Key Points

-  The `update` resource is renamed `task`. The names of existing API routes are also changed to reflect this change.
-  The format of the `task` object is updated.
    - `updateId` becomes `uid`.
    - Attributes of an error appearing in a `failed` `task` are now contained in a dedicated `error` object.
    - `type` is no longer an object. It now becomes a string containing the values of its `name` field previously defined in the `type` object.
    - The possible values for the `type` field are reworked to be more clear and consistent with our naming rules.
    - A `details` object is added to contain specific information related to a `task` payload that was previously displayed in the `type` nested object.
    - An `indexUid` field is added to give information about the related index on which the task is performed.
    - `processed` status changes to `succeeded`.
    - `startedProcessingAt` is updated to `startedAt`.
    - `processedAt` is updated to `finishedAt`.
- `202 Accepted` requests previously returning an `updateId` are now returning a summarized `task` object.

### II. Motivation

The main motivation is to stabilize the current `update` resource to a version that conforms to our API convention and thus allow future evolutions on a more solid base. We would like to modify the name `update`, its format is also to be reviewed because some attributes are not immediately clear, either in the possible values or in the chosen names.

### III. Explanation
TBD


### IV. Finalized Key Changes

## 2. Technical details

### I. Measuring

TBD

## 3. Future Possibilities

- Add pagination on `task` lists.
- Add filtering capabilities.
