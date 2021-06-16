- Title: Phrase Query
- Start Date: 2021-06-02
- Specification PR: [#43](https://github.com/meilisearch/specifications/pull/43)
- MeiliSearch Tracking-Issues: [transplant/#198](https://github.com/meilisearch/transplant/issues/198)

# Phrase Query

## 1. Feature Description and Interaction

### I. Summary

MeiliSearch does not allow users a way to write a strict query in order to ask the engine to be more strict in its selection of candidates for search results. The Phrase Query feature adds a simple syntax available to users to require the engine to select documents that strictly match some phrase, indicated by quotation marks. That is, without typography, n-gram, wordsplit, prefix, synonym, and, proximity. In addition, the expression of a Phrase Query is case insensitive.

### II. Motivation

This feature is driven by the user's needs. Indeed, let's take some examples recently brought up in our community Slack.

#### Example 1

The user wants to have a way to ensure that the words of his search are exactly contained in the order of the terms in the documents returned by MeiliSearch.

#### Example 2

The user in question would like to be able to retrieve specifically the document containing the unique ISBN identifier and only that one. In a UX context of type as you search, this is impossible today without impacting the UI/UX or finding a workaround.

- It could have had a specific search field only dedicated to this ISBN field in order to make it a filter and using the `=` operator. Not great for the UX.

- It could also have kept a single search field and detected that an ISBN was entered in the search field using a regex to inject that filter at that point. Not great for the developper experience, moreover, what happens when a pattern cannot be specifically determined?

The Phrase Query feature will easily solve the last case but will also adapt to the needs of the user performing the search.

### III. Additional Materials

#### Algolia

Algolia allows the use of Phrase Query syntax as long as the `advancedSyntax` parameter is set to true in the settings.

As Algolia documentation said, a phrase query represents a specific sequence of terms that must be matched next to one another and in the given order. A phrase query needs to be surrounded by double quotes ("). For example, the query "search engine" only returns a record if it contains “search engine” exactly in at least one attribute.

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

Let's say I want to search for a specific book with a title strictly containing `Plays and Playwrights 2002`.

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

#### Phrase Query usage

To use the Phrase Query syntax, simply surround the contiguous search terms with the characters `"`.

Using the Phrase Query syntax this way, with `q` equals to `"Plays and Playwrights 2002"`, will lead to have only one result because the title is written exactly like that.

> Note that it's case insensitive. So, if I search with `"plays and playwrights 2002"`, this will lead to the same result.

The value between the `"` operators will be searched without:

- typography
- n-gram
- wordsplit
- prefix
- synonym
- proximity

Moreover, it ensures that all matching documents contain the words in the given order in the Phrase.

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

So now let's say that I want to search for a title that strictly includes the phrase `"African American"` but speaking about poetry. The Phrase Query syntax can be used in conjunction with the basic syntax.

The query can be expressed like this: `"African American" poem`

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

As you can see in the results, the presence or absence of one or more soft separators such as `-`, `_`, `\`, `:`, `/`, `\\`, `@`, `"`, `+`, `~`, `=`, `^`, `*`, `#` between two words does not affect the query match of the phrase and the document.

#### Multiple Phrase Query usage

It is possible to use multiple phrase queries within a search.

E.g. `"African American" "Anthology"`

With this expression, the returned documents will contain exactly the existence of both phrase queries.

#### Know limitations

##### Case Sensivity

The Phrase Query syntax is case insensitive.

##### Multiple hard separator case

Given a document containing `David.- .- .- .- .-Bowie` as value for an attribute.

This document can be matched using a phrase query such as `"David" "Bowie"` or `"David.Bowie"`. At the engine level, this is the same query. The latter is translated into the former phrase because the `.` character is a hard separator. This behavior comes from the default tokenizer, hard separators are seen as a marker for a different context or phrase.

Here is the list of hard separators in the default tokenizer: `.`, `;`, `,`, `!`, `?`, `(`, `)`, `[`, `]`, `{`, `}`, `|`

Multiple hard separator are treaten the same as if they were one. So `"David.-.-.-.Bowie"` will not work to match the document.

### V. Impact on Documentation

- Mention that new Phrase Query syntax in the documentation. Can it be on a dedicated page or on [the q search parameter](https://docs.meilisearch.com/reference/features/search_parameters.html#query-q)?

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities
N/A
