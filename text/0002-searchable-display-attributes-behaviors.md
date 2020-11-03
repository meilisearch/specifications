- Title: Searchable Attributes and Displayed Attributes behaviors
- Start Date: 2020-11-02
- specification PR: meilisearch/specifications#3
- Meilisearch Issue: 

# `searchable_attributes` & `displayed_attributes` behaviors

## First section: Feature Description and Interaction

This first part has a general audience. It should be as little technical as possible (think user-level). This section contains 4 sub-sections:

### Summary

We wanted to change `searchable_attributes`, beacause it was confusing. We recently add wildcard in this field to select all. This last change introduced lot of edge-cases:
- we don't know the order of searchable attributes when we use wildcards.

Searchable attributes have several uses:
- know in which fields we can search.
- know the importance of fields for the criterion `Attributes`.

When a wildcard is used, the order of searchable attributes is written no-where.

Is it relevant to mix this 2 information when we have wildcards?
### Explanation

#### default values

`searchable_attributes`: *
`displayed_attributes`: *

#### wildcard (*) & `searchable_attributes` order

> When the searchable attributes value is set to ["*"] the priority of the documents attributes is undefined.

By default searchable attributes should be in the order of the first indexed document, for example:
```json
{
  "title": "many the fish",
  "description": "it is awfull to be liked by many"
}
```
default `searchable_attributes` order should be `["title", "description"]` where `"title" > "description"`

When a wildcard is set we should have the default behavior.

#### `searchable_attributes` & `displayed_attributes`

what is the behavior for a field that:
- > **is in `searchable_attributes` and in `displayed_attributes`:**
  > meilisearch will use the field for a search and the field will be returned in search results
- > **is in `searchable_attributes` but not in `displayed_attributes`:**
  > meilisearch will use the field for a search but the field will **not** be returned in search results
- > **is not in `searchable_attributes` but in `displayed_attributes`:**
  > meilisearch will **not** use the field for a search but the field will be returned in search results
- > **is not in `searchable_attributes` and not in `displayed_attributes`:**
  > meilisearch will **not** use the field for a search and the field will **not** be returned in search results. The field is unused but should be kept in case of it is added in `searchable_attributes` or `displayed_attributes`.

#### add a new configuration `attributes_order`?

This new config may define the order of attributes for the criterion `Attributes`, no wildcard allowed.
if `searchable_attribute` is: `["title", "desc"]`
and `attributes_order` is: `["desc", "id", "title"]`
The searchable attributes order would be:
`"desc" > "title"` (`"id"` is not searchable so is ignored by the criterion `Attributes`).

`attributes_order` is always exhaustive, If a new field is added, it is "pushed" at the end of the list.

##### drawbacks:
- new config added
- more complexity
- could be replace by a small note about wildcard in `searchable_attributes`

#### Changes in meilisearch documentation

Depending on what we choose, we'll have to add **default** and **wildcard** behavior of configurations, we'll potentially add a new configuration `attributes_order`. 

<!-- Explain the proposal as if it was already included in MeiliSearch and you were teaching it to another user. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- The API for this feature, HTTP, CLI or config.
- Explaining how the user should think about the feature and how it should impact the way they use MeiliSearch. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.

If the changes modify the HTTP API, provide a description of the method, URL, parameters, body, status code, errors, etc...

If it modifies the CLI, provide the env variable name, the argument name, and the description.

This serves as a user-level guide. Anything that the user may encounter during its interaction with the feature should be presented here.
Impact on documentation

If the feature requires additions to the documentation or if sections of the documentation need to be updated because of this feature, it should be mentioned here. It's the role of the documentation team to point out the sections of the documentation that need to be updated. -->

## Second Section: Technical Specifications

<!-- This section has a much narrower audience: the developer that will implement the feature. Its goal is to make it as clear as possible to develop the feature, share knowledge, and think about the possibilities. -->

### Architecture

This change will impact **settings**, the **indexer** and the **search-engine** (`Attribute` criterion).

### Implementation Details

> TBD

<!-- Some aspects will need to be made precise, such as interfaces or specific algorithmic choices. -->

### Corner Cases

If we change searchable attributes and after we re-add wildcard, what should be the order? undefined?

<!-- Some aspects of the development will necessitate special care, they should be pointed out, and if there are still unanswered questions, they belong here too. -->

## Third Section: Future possibilities

<!-- This last section talks about what has been thought of related to this issue, but has been decided not to be done now, and what it means regarding the feature at hand. -->
