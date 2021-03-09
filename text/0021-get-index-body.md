- Title: Index Response Body
- Specification PR: #21

# Index Response Body

## 1. Feature Description and Interaction

### Summary

As of today the following fields are present in the index body: 
```
{
    "name": "indexUID",
    "uid": "indexUID",
    "createdAt": "2021-03-02T14:45:17.837494800Z",
    "updatedAt": "2021-03-02T14:45:17.838545700Z",
    "primaryKey": null
}
```

Should fields be added and/or removed?

### Motivation


An `uuid` is generated in the new alpha version, adding a new field in the index body: 

```
{
  "uid": "indexUID",
  "uuid": "72114596-0807-4af4-9610-07fc2c957990",
  "createdAt": "2021-03-02T20:37:06.772799900Z",
  "updatedAt": "2021-03-02T20:37:06.772799900Z",
  "primaryKey": null
}
```

This field may be confusing as `uid` already exists. Should we keep both, rename or remove?

### Additional Materials

https://github.com/meilisearch/transplant/issues/67

### Explanation

### Impact on Documentation

## 2. Technical Specifications

### Architecture
### Implementation Details
### Corner Cases

## 3. Future Possibilities

- Add `uuid`
- Remove `uuid`
- Rename `uid`