- Title: Phrase Query
- Start Date: 2021-06-02
- Specification PR: [#43](https://github.com/meilisearch/specifications/pull/43)
- MeiliSearch Tracking-Issues:

# Phrase Query

## 1. Feature Description and Interaction

### I. Summary

Meilisearch does not allow users a way to write a strict query in order to ask the engine to be more drastic in its selection of candidates for search results. The Phrase Query feature adds a simple syntax available to users to require the engine to select documents that will strictly match the search field value. That is, without typography, without prefix and, in the order of the given terms.

### II. Motivation

This feature is driven by user needs. Indeed, let's take an example recently brought up in our community slack.

The user in question would like to be able to retrieve specifically the document containing the unique ISBN identifier and only that one. In an ux context of type as you search, this is impossible today without impacting the UI/UX or finding a workaround.

- It could have had a specific search field only dedicated to this ISBN field in order to make it a filter and using the `=` operator. Not great for the ux.

- It could also have kept a single search field and detected that an ISBN was entered in the search field using a regex to inject that filter at that point. Not great for the developper experience, moreover, what happens when a pattern cannot be specifically determined?

The Phrase Query feature will easily solve this case but will also adapt to the needs of the user performing the search.

### III. Additional Materials

#### Algolia

Algolia allows the use of Phrase Query syntax as long as the `advancedSyntax` parameter is set to true in the settings.

As Algolia documentation said, a phrase query represents a specific sequence of terms that must be matched next to one another. A phrase query needs to be surrounded by double quotes ("). For example, the query "search engine" only returns a record if it contains “search engine” exactly in at least one attribute.

#### Elasticsearch

Elasticsearch provides a `match_phrase` field to perform this type of search.

```
GET /_search
{
  "query": {
    "match_phrase": {
      "message": "this is a test"
    }
  }
}
```

### IV. Explanation

To use the Phrase Query syntax, simply surround the contiguous search terms with the characters `"`.

Let's say i want to search for a specific book with a title stricly containing `Plays and Playwrights 2002`.

Using the standard query parameter syntax as `q` equals to `Plays and Playwrights 2002`, will lead to have multiple search results because of the ranking rules criterions.

```
{
    "hits": [
        {
            "title": "Plays and Playwrights 2002",
            "author": "Martin Denton"
        },
        {
            "title": "Plays and Playwrights 2009",
            "author": "Martin Denton"
        },
        {
            "title": "Plays and Playwrights 2008",
            "author": "Robert Attweiler"
        },
        ...
        {
            "title": "The Best American Short Plays 2006-2007",
            "author": "Barbara Parisi"
        },
        {
            "title": "New Playwrights: The Best Plays of 2000",
            "author": "D. L. Lepidus"
        },
        {
            "title": "Women Heroes: Six Short Plays from the Women's Project",
            "author": "Julia Miles"
        }
    ],
    "nbHits": 31,
    "exhaustiveNbHits": false,
    "query": "Plays and Playwrights 2002",
    "limit": 20,
    "offset": 0,
    "processingTimeMs": 7
}
```

Using the Phrase Query syntax this way, with `q` equals to `"Plays and Playwrights 2002"`, will lead to have only one result because the title is exactly written like that. No typo tolerance, prefix search were applied on the query terms at search time.

```
{
    "hits": [
        {
            "title": "Plays and Playwrights 2002",
            "author": "Martin Denton"
        }
    ],
    "nbHits": 1,
    "exhaustiveNbHits": false,
    "query": "\"Plays and Playwrights 2002\"",
    "limit": 20,
    "offset": 0,
    "processingTimeMs": 0
}
```

So now let's say that i want to search title strictly mentioning `"African American"` but speaking about poetry. The Phrase query syntax can be used in conjunction with the basic syntax.

The query can be exprimed like that `"African American" poem`

```
{
    "hits": [
        {
            "title": "100 Best African American Poems with CD",
            "author": "Nikki Giovanni"
        },
        {
            "title": "The African American Experience: Black History and Culture Through Speeches, Letters, Editorials, Poems, Songs, and Stories",
            "author": "Kai Wright"
        },
        {
            "title": "African American Literature (Penguin Academics Series)",
            "author": "Keith Gilyard"
        },
        {
            "title": "African-American Poetry: An Anthology, 1773-1930",
            "author": "Joan R. Sherman"
        },
        {
            "title": "African-American Literature: A Brief Introduction and Anthology",
            "author": "Al Young"
        },
        {
            "title": "Early African American Classics (Barnes &amp; Noble Classics Series)",
            "author": "Barnes &amp; Noble"
        },
```

### V. Impact on Documentation

- Mention that new Phrase Query syntax in the [documentation](https://docs.meilisearch.com/reference/features/search_parameters.html#query-q).

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities
N/A

