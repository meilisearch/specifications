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
|   0   |   1   |   2   |
|   计      算      所   |
|<--------------------->|
|      3 char word      |
|<------------->|<----->|
|  2 char word  |1 char word
```
Which can be tokenized as `(word, start_pos, end_pos)` like:   
`[("计算所", 0, 3), ("计算", 0, 2), ("所", 2, 3)]`.

Taking an other example `永和服装饰品有限公司meilisearch`
this text has several equivalent derivation of tokenization:
```
|   0   |   1   |   2   |   3   |   4   |   5   |   6   |   7   |   8   |   9   |   ..        |
|   永      和      服       装      饰      品       有      限      公       司    meilisearch |
|<------------->|<------------->|<------------->|<------------->|<------------->|<----------->|
|  2 char word  |  2 char word  |  2 char word  |  2 char word  |  2 char word  | latin word  |
|                                               |<----------------------------->|             |
|                                               |          4 char word          |             |
```
Which can be tokenized as `(word, start_pos, end_pos)` like:   
`[("永和", 0, 2), ("服装", 2, 4), ("饰品", 4, 6), ("有限公司", 6, 10), ("有限", 6, 8), ("公司", 8, 10), ("meilisearch", 10, 21)]`.

_BUT, What should be the word position of each word?_

#### First Approach: Have all derivations at the same position

In this Approach, each derivations of the previous text `计算所` would be tokenized as `(word, word_position)` like:
`[("计算所", 0), ("计算", 0), ("所", 0)]`.   
Making searches `"计算所"`, `"计算"` or `"所"` match words at `word_position=0` perfectly.

Taking the other example `永和服装饰品有限公司meilisearch`:
`[("永和", 0), ("服装", 1), ("饰品", 2), ("有限公司", 3), ("有限", 3), ("公司", 3), ("meilisearch", 4)]`   
On the second example we can see some specificities:
- the word `("有限", 3)` is considered near `("meilisearch", 4)`, if we search `有限meilisearch` it would match perfectly
- the word `("饰品", 2)` is considered near `("公司", 3)`, if we search `饰品公司` it would match perfectly

#### Second Approach: Shift word position when derivations are possible

In this Approach, each derivations of the previous text `计算所` would be tokenized as `(word, word_position)` like:
`[("计算所", 0), ("计算", 0), ("所", 1)]`.   
Making searches `"计算所"` or `"计算"` match words at `word_position=0` perfectly; and making search `"所"` match word at `word_position=1` perfectly.

Taking the other example `永和服装饰品有限公司meilisearch`:
`[("永和", 0), ("服装", 1), ("饰品", 2), ("有限公司", 3), ("有限", 3), ("公司", 4), ("meilisearch", 5)]`   
On the second example we can see some specificities:
- the word `("有限公司", 3)` is considered far from `("meilisearch", 5)`, if we search `有限公司meilisearch` it would not match with perfect distance, this distance would be minimal and no other document could have a better distance but if some words have lot of derivations, this shifting could add significative noise in word proximity calculation.
- the word `("有限公司", 3)` is considered near `("公司", 4)`, this relation doesn't exist in the original document, and so making search `有限公司公司` could match perfectly the document.

#### BIG WARNING / HELP WANTED
We don't speak any Chinese language and we are just infering behaviors from `jieba` examples,
we probably miss something and the opinion of a real Chinese would help us to choose the best behavior for the Chinese Script Tokenizer.

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
