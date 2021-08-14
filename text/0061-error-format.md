- Title: Error Format
- Start Date: 2021-08-13
- Specification PR: [#61](https://github.com/meilisearch/specifications/pull/61)
- Discovery Issue: [#44](https://github.com/meilisearch/product/issues/44#issuecomment-896121211)

# Error Format

## 1. Functional Specification

### I. Summary

The error format is updated to no longer contain unnecessary `error` prefix terms in the names of these attributes. e.g. `errorType`. The context is clear enough to understand that this is an `error` object.

#### Summary Key Points

- `error` is removed from the attributes name.

### II. Motivation

The main motivation is to stabilize the current `error` resource to a version that conforms to our API convention and thus allows future evolutions on a more solid base. This specification avoids adding unnecessary information in the error object's attribute names.

### III. Explanation

#### Error Format

##### Attributes

| Field name | type   | Description                                                              |
|------------|--------|--------------------------------------------------------------------------|
| message    | string |  A human-readable message providing context and details about the error. |
| code       | string |  A short string indicating the error code reported.                      |
| type       | string |  The type of error returned.                                             |
| link       | string |  An URL to the related error-page details for further information.       |

##### Json Response Example

e.g. 401 Unauthorized Response example

```
{
    "message": "authorization header is missing",
    "code": "missing_authorization_header",
    "type": "authentication_error",
    "link": "https://docs.meilisearch.com/errors#missing_authorization_header"
}
```

## 2. Technical details
N/A

## 3. Future Possibilities

- Add a `parameter` attribute in the `error` object to indicate which parameter of the request caused an error.
