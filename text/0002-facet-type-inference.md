- Title: Facet Type Inference
- Specification PR:

# Facet Type Inference

## 1. Feature Description and Interaction

### Summary

Contrary to MeiliSearch v0.20.0, the search engine that will be released in the v0.21.0 needs to know the type of the faceted fields.<br>
Once the fields are set to `attributesForFaceting`, they can be used during the search as filters or to know the facet distributions.

We want MeiliSearch to infer the type of the fields set to `attributesForFaceting` and also let the possibility of the users set the type they want if they don't want the inference.

### Motivation

The inference avoids losing user experience.

### Additional Materials

Algolia does not require defining the type of the faceted fields, that's also why we want to implement type inference.

### Explanation

#### Possible data types

- `string`
- `number`

Number = float or integer.
ex: `12` or `12.04`

Every "number" (integer or float) field is converted into float.
⚠️ Integers with high values lose precision. However, integers from −2^53 to 2^53 (−9007199254740992 to 9007199254740992) can be exactly represented, which is enough in 99% of cases.

#### Settings push

When updating the `attributesForFaceting`:

- Types are inferred

```json
{
    "attributesForFaceting": [
        "price",
        "author"
    ]
}
```

- Types are specified

```json
{
    "attributesForFaceting": [
        { "price": "number" },
        { "author": "string" }
    ]
}
```

- Both, typed and inferred

```json
{
    "attributesForFaceting": [
        { "price": "number" },
        "author"
    ]
}
```

#### Settings are set before documents are indexed (inference impossible)

The users push the fields in `attributesForFaceting` (using inference):

```json
{
    "attributesForFaceting": [
        "price",
        "author"
    ]
}
```

But they did not push the documents before => This is impossible for MeiliSearch to guess the type, so the behavior is `undefined`.

Concretely, if they try to get the settings, the display is:

```json
{
    "attributesForFaceting": [
        { "price": "undefined" },
        { "author": "undefined" }
    ]
}
```

RESTful behavior: if there is not type already defined for a field, the user can push to the settings the `undefined` type (same as just the attribute in a string). If there is already a type for the field, the `undefined` is ignored.
This RESTful behavior should not be documented.

#### Inference behavior

The inference behavior should not be documented. Why? Under the hood, the first document in MeiliSearch (but not necessarily the first one sent) is used for the inference. It's impossible for the user to know what is this first document in MeiliSearch, so impossible for the user to know what document is used for the inference.

If the user does not pass any type in `attributesForFaceting`, MeiliSearch has to infer

- JSON string (ex: `"kiki"`) -> `string`
- JSON number (ex: `12`, `12.02`) -> `number`
- JSON boolean -> `string`
- JSON `null` -> `undefined`
- JSON array -> type of the first element. If it's an array of arrays (or objects...), the type is `undefined`.
- JSON object -> `undefined`

=> should this behavior be documented?

#### Convertion behavior

If the users set a field as a `string` but the same field contains another type, like `number`, MeiliSearch has to convert it.

If MeiliSearch cannot convert (ex: convert "titi" into `number`), it returns a warning in the logs and the document is accepted. During the filtering, the non-converted document will be ignored.

Convertion list:

- `number` into `string` -> trying to convert, depending on the value (cf description above)
- `string` into `number`-> trying to convert, depending on the value (cf description above)
- boolean into `string` -> "true"/"false"
- boolean into `number` -> 1/0
- null into `number`/`string` -> document accepted but ignored during the filtering
- object into `number`/`string` -> document accepted but ignored during the filtering
- array into `number`/`string` -> depends on the element of the array (1-depth max)

=> should this behavior be documented?

#### Error during the search

- `price` as string in settings
- search `filters: "price < 12"`

=> Should return an error during the search "Impossible to apply a numeric operation on string field (price)"

### Impact on Documentation

// TODO

## 2. Technical Specifications

// TODO

### Architecture
### Implementation Details
### Corner Cases

## 3. Future Possibilities

In the future, we want to remove this inference by searching in the "string database" and in the "numeric database" at the same time without relevancy or performance issues.
