- Title: Data Types
- Start Date: 2022-03-02

# Data Types

## 1. Summary

This specification describes the different data types supported for the fields in a document and how Meilisearch handles them.

## 2. Functional Specification

No matter the type, the value of a field is unchanged in the returned documents upon search.

For example, if you have a complex field with nested objects, this field is returned with the same complexity upon search.

Based on their type, however, the fields are handled and used in different ways by Meilisearch. The type affects how a field is used for search results.

### 2.1. Supported types

| Type    |
|---------|
| String  |
| Numeric |
| Boolean |
| Array   |
| Object  |
| null    |

#### 2.1.1. String

`String` is the primary type for indexing data in Meilisearch. It enables to create the content in which to search.

> See 3.1. String Tokenization section.

#### 2.1.2. Numeric

A numeric type (`integer`, `float`) is internally converted to a human-readable decimal number string representation. Numeric types can be searched as they are converted to strings.

The `>`, `>=`, `<`, and `<=` opearators of the `filter` search parameter apply only to numerical values.

#### 2.1.3. Boolean

A `Boolean` value, which is either `true` or `false`, is received and converted to a lowercase human-readable text (i.e. `true` and `false`). Booleans can be searched as they are converted to strings.

#### 2.1.4. Array

An array represents a collection of elements that can be strings or arrays for instance. An array is recursively broken into separate string tokens, which means separate words.

After the tokenizing process, each word is indexed and stored in the global dictionary of the corresponding index.

> See 3.2. Array Tokenization section.

Meilisearch accepts complex data structures, no matter the deepness level.

> See 3.3. Nested structures section.

#### 2.1.5 Object

JSON objects are written in key/value pairs and surrounded by curly braces. Internally, an object is flattened at the root level of a document.

After the tokenizing process, each word is indexed and stored in the global dictionary of the corresponding index.

Meilisearch accepts complex data structures, no matter the deepness level.

> See 3.3. Nested structures section.

## 3. Technical Details

### 3.1. String Tokenization

String tokenization is the process of splitting a string into a list of individual terms that are called tokens.

A string is passed to a tokenizer and is then broken into separate string tokens. A token is a word.

For Latin-based languages, the words are separated by space.
For Kanji characters, the words are separated by character.
For Latin-based languages, there are two kinds of space separators:

- **Soft spaces (distance: 1)**: whitespaces, quotes, `-` | `_` | `\` | `:` | `/` | `\\` | `@` | `"` | `+` | `~` | `=` | `^` | `*` | `#`
- **Hard spaces (distance: 8)**: `.` | `;` | `,` | `!` | `?` | `(` | `)` | `[` | `]` | `{` | `}`| `|`

Distance plays an essential role in determining whether documents are relevant since one of the ranking rules is the proximity rule. The proximity rule sorts the results by increasing distance between matched query terms. Then, two words separated by a soft space are closer and thus considered more relevant than two words separated by a hard space.

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

#### 3.2.1 Examples

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

> See 3.1.1. Examples section.


### 3.2 Nested Structures

Nested structures (e.g. `Object`, `Array of Objects, etc) are internally flattened to a document's root level.

It allows expressing a nested field in all Meilisearch parameters that accept document attributes.

Meilisearch accepts the `.` notation to express a nested field location.

> See 3.2.1 Examples section.

### 3.2.1 Examples

#### 3.2.1.1. Object

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

#### 3.2.1.2. Array of objects

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

#### 3.2.1.3. Array of objects mixed with scalar value

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


#### 3.2.1.4. Array of objects of array of objects of ...

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

#### 3.2.1.5. Collision between a representation

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

## 4. Future Possibilities
n/a