# Version API

## 1. Summary

This specification describes the version API endpoint. The `/version` route allows to check the version of a running Meilisearch instance.

## 2. Motivation

Since users don't always have SSH access at hand, it can be useful to give information about the version concerned when they encounter a bug or a problem.

## 3. Functional Specification

### 3.1. `version` API resource properties

| Field                                            | Type            | Required |
|--------------------------------------------------|-----------------|----------|
| [commitSha](#311-commitSha)                      | String          | True     |
| [commitDate](#312-commitDate)                    | String          | True     |
| [pkgVersion](#313-pkgVersion)                    | String          | True     |

#### 3.1.1. `commitSHa`

- Type: String
- Required: True

The git commit identifier that tagged this release version number.

#### 3.1.2. `commitDate`

- Type: String
- Required: True

The date when the version tag has been created.

#### 3.1.3. `pkgVersion`

- Type: String
- Required: True

The Meilisearch binary version number.

## 3.2. API Endpoints Definition

### 3.2.1. `GET` - `version`

Retrieves the version information of the Meilisearch binary.

`200` - Response body

```json
    {
        "commitSha": "b46889b5f0f2f8b91438a08a358ba8f05fc09fc1",
        "commitDate": "2019-11-15T09:51:54.278247+00:00",
        "pkgVersion": "0.1.1"
    }
```

All properties must be returned when the resource is retrieved.

### 3.2.2. General Errors

These errors apply to all endpoints described here.

#### 3.2.2.1. Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have  the `version` action as a permission returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 2. Technical Details

n/a

## 3. Future Possibilities

n/a