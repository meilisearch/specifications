- Title: Chinese Script Tokenization
- Start Date: 2020-11-04
- specification PR:
- Meilisearch Issue: 

# Chinese Script Tokenization

## First section: Feature Description and Interaction

### Summary

In Chinese Script, there can be multiple "words" extracted that are considered to be at the same position in the text. For example 计算所 gives 计算 and 计算所, that are at the same position in the text but don't have the same length, the tokenizer should support that behavior.


### Motivation

Have a better tokenizer for Chinese Script will enhance the relevancy of search for chinese users.

### Prior Art and R&D
### Explanation

Meilisearch is based on word positions (the index of the object Word in an array of Word.s) and not on the index of the first and the last character. This is an issue when we want to handle Scripts that have several derivations of the same word like Chinese.

Taking the example of `计算所`,
this text has several equivalent derivation of tokenization:
```
|   1   |   2   |   3   |
|计         算         所|
|<--------------------->|
|      3 char word      |
|<------------->|    |<>|
|  2 char word  |    |1 char word
```
Which can be tokenized as `(word, start_pos, end_pos)` like:   
`[("计算所", 0, 3), ("计算", 0, 2), ("所", 2, 3)]`.

_BUT, What should be the word position of each word?_

#### First Approach: Have all derivations at the same position

TBD: @many

#### Second Approach: Shift word position when derivations are possible

TBD: @many

### Impact on documentation

None

## Second Section: Technical Specifications

### Architecture

This tokenizer will extend the meilisearch-tokenizer library specified by [001-script-based-tokenizer](https://github.com/meilisearch/specifications/pull/2) (change link when PR is merged),
and has a low impact on others parts of meilisearch.

### Implementation Details

TBD: @many @mpostma

### Corner Cases

TBD: @many @mpostma

## Third Section: Future possibilities
