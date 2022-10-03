# Health API

## 1. Summary

This specification describes the health API endpoint. The `/health` route allows to verify the status and availability of a Meilisearch instance.

## 2. Motivation

To know the status of a Meilisearch instance, `/health` is a public route that can be long pooled with simple monitoring tools.

## 3. Functional Specification

### 3.1. `health` API resource properties

| Field                                            | Type            | Required |
|--------------------------------------------------|-----------------|----------|
| [status](#311-status)                            | String          | True     |

#### 3.1.1. `status`

- Type: String
- Required: True
- Default: `available`

Returns the status of a Meilisearch instance, the only possible value is `available`.

## 3.2. API Endpoints Definition

### 3.2.1. `GET` - `health`

Retrieves the status of a Meilisearch instance.

`200` - Response body

```json
    {
        "status": "available"
    }
```

All properties must be returned when the resource is retrieved.

### 3.2.2. Endpoint access

This route is always public and access to it cannot be protected by an API key.

## 2. Technical Details

n/a

## 3. Future Possibilities

n/a