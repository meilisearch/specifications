- Title: Task Resource Fields Selector
- Start Date: 2021-09-14
- Specification PR: [#76](https://github.com/meilisearch/specifications/pull/76)

# Task Resource Fields Selector

## 1. Functional Specification

### I. Summary

Add fields selection capabilities for the `task` resource to facilitate reading and retrieving of informations with the use of sparse fieldset.

#### Summary Key Points

- Add a `fields` query parameter to select attributes to return in `task` objects.

### II. Motivation

Following the specification aiming to stabilize the `task` API resource, we want to give users the capability to request a sparse fieldset for the `task` resource.

### III. Technical Explanations

#### Query parameter definition

| parameter | type | required | description           |
|--------|--------|----------|----------------------------|
| fields | string | No | Default to all `task` object fields. Refer to the [Fullly Qualified Task Fieldset Definition](https://github.com/meilisearch/specifications/blob/bcfa888116f1b075f4b7313a516eb851812bbddc/text/0060-refashion-updates-apis.md#fully-qualified-task-object). `-attributeName` removes the field from the `task` object while `attribute` adds the field to the `task` object. Attributes can be separated by `,` character. |

### Usages examples

This section demonstrates fields selection for the `task` resource.

---

**No fields selection - Default behavior**

`GET` - `/tasks`

```json
{
    "results": [
        {
            "uid": 0,
            "indexUid": "movies",
            "status": "succeeded",
            "type": "settingsUpdate",
            "details": {
                "rankingRules": [
                    "typo",
                    "ranking:desc",
                    "words",
                    "proximity",
                    "attribute",
                    "exactness"
                ]
            },
            "duration": "PT1S",
            "enqueuedAt": "2021-08-10T14:29:17.000000Z",
            "startedAt": "2021-08-10T14:29:18.000000Z",
            "finishedAt": "2021-08-10T14:29:19.000000Z"
        },
        ...
    ],
    ...
}
```

**Fields selection to request a sparse fieldset**

`GET` - `/tasks?fields=uid,status,type,finishedAt`

```json
{
    "results": [
        {
            "uid": 0,
            "status": "succeeded",
            "type": "settingsUpdate",
            "finishedAt": "2021-08-10T14:29:19.000000Z"
        },
        ...
    ],
    ...
}
```

**Fields removing to create a sparse fieldset**

`GET` - `/tasks?fields=-enqueuedAt,-startedAt,-details`

```json
{
    "results": [
        {
            "uid": 0,
            "indexUid": "movies",
            "status": "succeeded",
            "type": "settingsUpdate",
            "duration": "PT1S",
            "finishedAt": "2021-08-10T14:29:19.000000Z"
        },
    ],
    ...
}
```

- 💡 As seen in the last examples specifying fields without the `-` operator selects only those fields while prefixing a field with`-` allows you to remove this field from the default format.

---

### Behaviors for `fields` query parameters.

- 💡 `fields` not set default to having the fullly qualified task fieldset definition.
- 💡 `fields` set to an empty value default to having the fullly qualified task fieldset definition.
- 💡 If `fields` contains an attribute that does not exists, the attribute is not taken into account, thus no error is raised.

## 2. Technical Aspects
n/a

## 3. Future Possibilities

- Use a HTTP header to propose different pre-formatted `task` object. e.g `summarized+task.json`, `complete+task.json`.
- Enhance it to make definition of a sparse fieldset on nested object among the `task` object. e.g. `fields=details.numberOfDocuments` or `fields[details]=numberOfDocuments`.