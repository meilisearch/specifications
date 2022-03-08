- Title: Data Types

# Data Types

## 1. Summary

This specification describes the different data types supported for the fields in a document and how Meilisearch handles them.

## 2. Functional Specification

No matter the type, the value of a field is unchanged in the returned documents upon search.

For example, if you have a complex document structure with nested objects, the document is returned with the same complexity upon search.

However, based on their type, the fields are handled and used in different ways by Meilisearch.

### 2.1. Supported types

- [String](#211-string)
- [Numeric](#212-numeric)
- [Boolean](#213-boolean)
- [Array](#214-array)
- [Object](#215-object)
- [null](#216-null)

#### 2.1.1. String

`string` is the primary type for indexing data in Meilisearch.

> See [3.1. String Tokenization section](#31-string-tokenization).

#### 2.1.2. Numeric

The engine internally converts a `numeric` typed value (`integer`/`float`) to a human-readable decimal number string representation to make them searchable.

The `>`, `>=`, `<`, and `<=` `filter` operators apply only to numerical values.

#### 2.1.3. Boolean

The engine internally converts a `boolean` typed value (`true`/`false`) to a lowercase human-readable text to make it searchable.

#### 2.1.4. Array

An array is recursively broken into separate string tokens, which means separate words.

After the tokenizing process, each word is indexed and stored in the global dictionary of the corresponding index.

> See [3.2. Array Tokenization section](#32-array-tokenization).

Meilisearch accepts complex data structures, no matter the deepness level.

> See [3.4. Nested structures section](#34-nested-structures).

#### 2.1.5. Object

The engine flattens JSON objects at the root level of a document.

After the tokenizing process, each word is indexed and stored in the global dictionary of the corresponding index.

> See [3.3. Object section](#33-object).

Meilisearch accepts complex data structures, no matter the deepness level.

> See [3.4. Nested structures section](#34-nested-structures).

#### 2.1.6. `null`

The `null` type is not taken into account at indexing.

## 3. Technical Details

### 3.1. String Tokenization

String tokenization is the process of splitting a string into a list of individual terms that are called tokens.

A string is passed to a tokenizer and is then broken into separate string tokens. A token is a word.

For Latin-based languages, the words are separated by space.
For Kanji characters, the words are separated by character.

For Latin-based languages, there are two kinds of space separators:
- **Soft spaces (distance: 1)**: whitespaces, quotes, `-` | `_` | `\` | `:` | `/` | `\\` | `@` | `"` | `+` | `~` | `=` | `^` | `*` | `#`
- **Hard spaces (distance: 8)**: `.` | `;` | `,` | `!` | `?` | `(` | `)` | `[` | `]` | `{` | `}`| `|`

Distance plays an essential role in determining whether documents are relevant. The `proximity` ranking rule sorts the results by increasing distance between matched query terms. Two words separated by a soft space are closer and thus considered more relevant than two words separated by a hard space.

After the tokenizing process, each word is indexed and stored in the global dictionary of the corresponding index.

#### 3.1.1. Examples

To demonstrate how a string is split by space, let's say you have the following string as an input:

`"Bruce Willis,Vin Diesel"`

In the example above, the distance between `Bruce` and `Willis` is equal to `1`. The distance between `Vin` and `Diesel` is equal to `1` too.
But, the distance between `Bruce` and `Vin` is equal to `8`. The same goes for `Bruce` and `Diesel`, or `Willis` and `Vin`, or also `Willis` and `Diesel`.

Let's see another example. Given two documents:

```json
[
    {
        "movie_id": "001",
        "description": "Bruce.Willis"
    },
    {
        "movie_id": "002",
        "description": "Bruce super Willis"
    }
]
```

When making a query on `Bruce Willis`, `002` will be the first document returned and `001` will be the second one.
This will happen because the proximity distance between `Bruce` and `Willis` is equal to `2` in the document `002` whereas the distance between `Bruce` and `Willis` is equal to `8` in the document `001` since the full stop is a hard space.

### 3.2. Array Tokenization

An array is recursively broken into separate string tokens, which means separate words. After the tokenizing process, each word is indexed and stored in the global dictionary of the corresponding index.

#### 3.2.1. Examples

The following input:

```json
[
    [
        "Bruce Willis",
        "Vin Diesel"
    ],
    "Kung Fu Panda"
]
```

Will be processed as if all elements were arranged at the same level:

```json
"Bruce Willis. Vin Diesel. Kung Fu Panda."
```

The strings above will be separated by soft and hard spaces exactly as explained in the string example.

> See [3.1.1. Examples section](#311-examples).

### 3.3. Object

#### 3.3.1. Example

The following input:

```json
{
    "movie_id": "1564saqw12ss",
    "title": "Kung fu Panda"
}
```

In the example above, `movie_id`, `1564saqw12ss`, `title`, `Kung fu Panda` are all considered sentences. The colon `:` and comma `,` characters are used as separators.

`"movie_id. 1564saqw12ss. title. Kung fu Panda."`

These sentences will be separated by soft and hard spaces precisely as explained in the string example.

### 3.4. Nested Structures

Nested structures (e.g. `Object`, `Array of Objects`, etc) are internally flattened to a document's root level.

It allows expressing a nested field in all Meilisearch parameters that accept document attributes.

Meilisearch accepts the `.` (dot-notation) to express a nested field location in a document structure.

### 3.4.1. Examples

#### 3.4.1.1. Object

The following JSON document:

```json
    {
        "a": {
            "b": "c",
            "d": "e",
            "f": "g"
        }
    }
```

Flattens to:

```json
{
    "a.b": "c",
    "a.d": "e",
    "a.f": "g"
}
```

#### 3.4.1.2. Array of objects

The following JSON document:

```json
{
    "a": [
        { "b": "c" },
        { "b": "d" },
        { "b": "e" },
    ]
}
```

Flattens to:

```json
{
    "a.b": ["c", "d", "e"],
}
```

#### 3.4.1.3. Array of objects mixed with scalar value

The following JSON document:

```json
{
    "a": [
        42,
        { "b": "c" },
        { "b": "d" },
        { "b": "e" },
    ]
}
```

Flattens to:

```json
{
    "a": 42,
    "a.b": ["c", "d", "e"],
}
```


#### 3.4.1.4. Array of objects of array of objects of ...

The following JSON document:

```json
{
    "a": [
        "b",
        [
            "c",
            "d"
        ],
        {
            "e": [
                "f",
                "g"
            ]
        },
        [
            {
                "h": "i"
            },
            {
                "e": [
                    "j",
                    {
                        "z": "y"
                    }
                ]
            },
        ],
        ["l"],
        "m"
    ]
}
```

Flattens to:

```json
{
    "a": ["b", "c", "d", "l", "m"],
    "a.e": ["f", "g", "j"],
    "a.h": "i",
    "a.e.z": "y"
}
```

#### 3.2.4.5. Collision between a representation

The following JSON document:

```json
{
    "a": {
        "b": "c",
    },
    "a.b": "d"
}

Flattens to:

```json
{
    "a.b": ["c", "d"],
}
```

#### 3.2.4.6. searchableAttributes default value case

By default, `searchableAttributes` is set to `[*]`, making all document fields are searchable.

In that case, `Attribute` ranking rule consider a field higher in the internal representation more important than a lower one.

Field order is lost if the engine flattens identical field values are not co-located in a document payload.

The following JSON document:

```json
{
    "a.b": "T-shirt",
    "price": 2.0,
    "a": {
        "b": "Nice T-shirt"
    }
}
```

Is internally represented

```json
{
    "a.b": ["T-shirt", "Nice T-shirt"],
    "price": 2.0
}
```

The second representation of `a.b` in its nested form is merged with the first representation of `a.b`.

Users can't and should not rely on a given document field order when `searchableAttributes` is `[*]`.

#### 3.2.4.7. Dot-notation expression

##### 3.2.4.7.1. Concerned API parameters

All Meilisearch parameters that accept document attributes support the dot-notation.

Here is an exhaustive list of parameters supporting this notation:

Index API

- `primaryKey`

Document API

- `attributesToRetrieve`
- `primaryKey`

Settings API

- `displayedAttributes`
- `searchableAttributes`
- `filterableAttributes`
- `sortableAttributes`
- `rankingRules` (Custom ranking rule declaration)
- `distinctAttribute`

Search API

- `attributesToRetrieve`
- `attributesToHighlight`
- `attributesToCrop`
- `filter`
- `sort`
- `facetsDistribution`

##### 3.2.4.7.2. Examples

###### 3.2.4.7.2.1 Expressing a precise nested field

Given this document structure

```json
{
    "id": 0,
    "person": {
        "firstname": "John",
        "lastname": "Doe",
        "address": {
            "country": "US",
            "city": "New York"
        }
    }
}
```

A precise field can be expressed using the dot-notation

```json
{
    "attributesToHighlight": ["person.firstname"]
}
```

###### 3.2.4.7.2.2 Expressing all object properties

Given this document structure

```json
{
    "id": 0,
    "person": {
        "firstname": "John",
        "lastname": "Doe",
        "address": {
            "country": "US",
            "city": "New York"
        }
    }
}
```

All properties of a document nested object can be expressed using the dot-notation

```json
{
    "attributesToHighlight": ["person"]
}
```

It's equivalent to

```json
{
    "attributesToHighlight": [
        "person.firstname",
        "person.lastname",
        "person.address.country",
        "person.address.city"
    ]
}
```

## 4. Future Possibilities

- Change the default behavior of `searchableAttributes` so that it is predictable. We may remove the priority based on a field position in a document.
- Support the wildcard notation with the dot-notation. e.g. `person.*` or `person.address.*`