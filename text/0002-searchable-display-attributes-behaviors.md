- Title: Searchable Attributes and Displayed Attributes behaviors
- Start Date: 2020-11-02
- specification PR: meilisearch/specifications#3
- Meilisearch Issue: 

# `searchable_attributes` & `displayed_attributes` behaviors

## First section: Feature Description and Interaction

### Summary

Ever since the beginning, the design of `searchableAttributes` and `displayedAttributes` as shown flaws. The introduction of the wildcard tried to address some of those flaws, but there are still issues that this specs tries to correct.

Searchable attributes have several uses:
- know in which fields we can search.
- know the importance of fields for the criterion `Attributes`.

The main issues we want to address here are:
- The question of `searchableAttributes` order when the wildcard is used.
- The fact that already known attributes (e.g from facets), are not indexed correctly is put before the documents (bug), need to be taken into account in the design.


### Motivation

The current behavior of `SearchableAttributes` is buggy and confusing, it is necessary that this setting is stabilized as soon as possible.

#### Related issues

- https://github.com/meilisearch/MeiliSearch/issues/1066
- https://github.com/meilisearch/MeiliSearch/issues/942
- 
### Prior Art and R&D

TODO

### Explanation

For simplicity sake, we want MeiliSearch to work without configuration. To achieve this, the expected behaviour of Meiilisearch is to consider all attributes to be both `displayedAttributes` and `SearchableAttributes` by default.
Currently, the `SearchabeAttributes` fields is order sensitive: the order of the fields in this array define the order of the fields in any documents, hence it's importance in a search with the attribute ranking rule. By default, this order is set to the order of the fields of the first document to be indexed.

The default values for both of these attributes is `["*"]` (wildcard), to symbolize that all fields are contained.

This way of doing thing comes with a few caveats, that need to be addressed here:

#### wildcard (*) & `searchable_attributes` order

The current choice of having an ordered array for `SearchableAttributes` means that when the value is set to wildcard (its default value), we loose all information about the order.

> When the searchable attributes value is set to ["*"] the priority of the documents attributes is undefined.

By default searchable attributes should be in the order of the first indexed document, for example:

```json
{
  "title": "many the fish",
  "description": "it is awfull to be liked by many"
}
```
The default `searchable_attributes` order should be `["title", "description"]` where `"title" > "description"`.

TBD: What happens if the order is changed and wildcard is set back to `*`?
- undefined behaviour: the order when using `*` can't be relied upon
- some default value: which? Why?
- Dissociate order and existence.

#### Pre-existing attributes

Another issue with the current functioning of the attributes resides in the way we register the attributes. If a user adds, for example, attributes to the `facetAttributes` before indexing any documents, then this field will be registered as a known field, and when the user adds eventually indexes his documents, the field previously registered is not add to the searchable and displayed attributes.

This means that there should be a distinction between the creation of an `fieldId` (when a field is first registered), and it's usage/position in `searchableAttributes` and `displayedAttributes`.
What that means is that when we index documents, we must check if a field already has a position, and if not, give it a position (after adding it to the fieldmap) according to its position in the document.

##### caveat
- If a new document contains a new field at a position for which we already have a field:
```json
[
	{
		"id": 1,
		"title": "the title"
	},
	{

		"id": 1,
		"titre": "le titre"
	}
]
```

- To make sure that the `searchableAttributes` and `displayedAttributes` are in sync, the must come from the same source of information; this should be the field_position.
 TBD: What should be the position of `titre`?
	- same position as `title` (thus allowing attributes to exists at the same position like Algolia)
	- added at the last position

#### `searchable_attributes` & `displayed_attributes`

Being searchable and displayed are 2 different concept, and there should not be interactions between the two: a field can be searchable and not displayed, or displayed and not searchable, etc.

#### add a new configuration `attributes_order`?

Adding a new configuration `attributesPosition` that link all attributes present in a document  to a position makes it possible to keep track of the position without caring about whether a field is searchable or displayed. This also allows to keep track of all the fields that have been found in documents in the database.

```json
{
	"id": 0,
	"title": 1,
	"titre": 1
}
```

The default order is then **always** the order a field was in a document when it was first found.

##### drawbacks:
- new config added
- could be replace by a small note about wildcard in `searchable_attributes`


### Changes in meilisearch documentation

Depending on what we choose, we'll have to add **default** and **wildcard** behavior of configurations, we'll potentially add a new configuration `attributes_order`. 

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
