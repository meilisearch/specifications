- Title: Script Based Tokenizer
- Start Date: 2020-10-27
- specification PR: meilisearch/specifications#2
- Meilisearch Issue: meilisearch/Meilsearch#624

## Feature Description and Interaction

This first part has a general audience. It should be as little technical as possible (think user-level). This section contains 4 sub-sections:

### Summary

Tokenizer's principal role is to split documents into words (tokens) so each document can be indexed by it's contained words. It is also needed to split a search query into tokens, and search in the words index.

### Motivation

It turns out that splitting texts into tokens is a complicated task, especially when you need to handle more than a single language. While the task for english may seem trivial at first glance (mostly split white space), Chinese tokenization, for example is completely different (no spaces between words, groups of 1, 2 or 3 characters...).

Actual Tokenizer is naive and partially handle Latin Scripts only, lot of users ask for a smarter tokenizer to handle their own languages.
We should implement a new tokenizer which detect script and tokenize based on it.

### Prior Art and R&D

> TO BE DIEFINED
<!-- Discuss prior art, both the good and the bad, concerning this proposal. Put some links about what we can see on other tools, search API, or dev tools.

This section intends to encourage you as an author to think about the lessons from other tools and provide readers of your RFC with a fuller picture. If there is no prior art, that is fine - your ideas are interesting to us, whether they are brand new or adaptation from other tools. -->

### Explanation

The tokenizer serves as a proxy for other tokenizers, specialized in the language detected by the Tokenizer. It is instantiated with a String, and is then polled for tokens until it's delepted:
```rust
let tokenizer = Tokenizer::new("The quick brown fox jumps over the lazy dog");
assert_eq!(tokenizer.next().txt(), "The")
assert_eq!(tokenizer.next().txt(), " ")
assert_eq!(tokenizer.next().txt(), "quick")
```

### Impact on documentation

This feature should not impact the meilisearch user documentation,
in future versions we will probably provide a way to configure tokenizer and this will be discussed in a new specification.

## Technical Specifications

This section has a much narrower audience: the developer that will implement the feature. Its goal is to make it as clear as possible to develop the feature, share knowledge, and think about the possibilities.

### Architecture

As the actual tokenizer, this tokenizer will be a standolone library,
and will be used by meilisearch like another one.
We decided to put it in an other repository to be able to use it in meilisearch/Meilisearch and in the future core-engine.

### Implementation Details

We could have 2 approach of tokenization based on the script,
the main difficulty is to handle multi-script documents
which need to segment each part of script and tokenize them differently,
theses 2 approach are:
- first iterate over the whole document, splitting it by script, and after, tokenize each part with specialized lexer
- detect the script of the start of the document, tokenize with specialized lexer while script is the same, switch lexer if the script change

#### Tokenizer

```rust
use crate::token::Token;
use crate::internal_tokenizer::InternalTokenizer;

struct Tokenizer<'a> {
    /// script specialized tokenizer, this can be switched during
    /// document tokenization if the document contains several scripts
    current_tokenizer: Option<Box<dyn InternalTokenizer<'a>>>,
    /// current character index in the document
    current_char_index: u64,
    /// reference on the document content
    inner: &'a str,
}

impl<'a> Tokenizer<'a> {
    /// create a new tokenizer detecting script
    /// and chose the specialized internal tokenizer
    fn new(inner: &'a str) -> Self { unimplemented!() }
}

impl<'a> Iterator for Tokenizer<'a> {
    type Item = Token<'a>;

    /// return the next token calling the `next` method of the internal tokenizer,
    /// if the internal tokeizer return None, the function try to redetect script and chose a new tokenizer,
    /// if no iternal tokenizer is chosen, the method return None
    fn next(&mut self) -> Option<Self::Item> { unimplemented!() }
}
```

#### Token

```rust
/// script of a token (https://docs.rs/whatlang/0.10.0/whatlang/enum.Script.html)
pub type Script = whatlang::Script;

/// atomic item returned by `Tokenizer::next()`
pub struct Token<'a> {
    word: Lexeme<'a>,
    script: Script,
}

impl<'a> Token<'a> {
    fn txt(&self) -> &str { unimplemented!() }
}

/// Determine the type of the token and contains a `WordSlice`
pub enum Lexeme<'a> {
    /// the token is a word,
    /// meaning that it should be indexed as an important part of the document
    Word(WordSlice<'a>),
    /// the token is a stop word,
    /// meaning that it can be ignored to optimize size and performance or be indexed as a Word
    StopWord(WordSlice<'a>),
    /// the token is a separator,
    /// meaning that it shouldn't be indexed but used to determine word proximity
    Separator(WordSlice<'a>)
}

/// The script, the char_index and the content of the token
pub struct WordSlice<'a> {
    /// content of the token
    pub word: &'a str,
    /// index of the first character of the token in the whole document
    pub char_index: usize,
}
```

#### Internal Tokenizer trait (Lexer)
The `InternalTokenizer` traits provides a common interface to adapt other tokenizers to the tokenizer. This allows extensability of the current tokenizer to other languages.
```rust
use crate::token::{Token, WordSlice, Script};

/// trait defining an internal tokenizer,
/// an internal tokenizer should be a script specialized tokenizer,
/// this should be implemented as an `Iterator` with `Token` as `Item`,
pub(crate) trait Lexer<'a>: Iterator<Item = Lexeme<'a>> {
    /// create the tokenizer based on the given `text` and `char_index`
    /// [ERR]: this trait cannot be made into an object because associated function `new` has no `self` parameter the trait `internal_tokenizer::InternalTokenizer` cannot be made into an object
    fn new(text: &'a str, char_index: usize) -> Self;
    /// return the `char_index` of the next potential token,
    /// it should be used when switching internal tokenizer
    /// to retrieve the tokenization state
    fn char_index(&self) -> usize;
}

/// return a box of the specialized internal tokenizer for the given `Script`,
/// calling the method new of the chosen internal tokenizer
pub(crate) fn from_script<'a>(script: Script, inner: &'a str, char_index: usize) -> Option<Box<dyn InternalTokenizer<'a>>> { unimplemented!() }
```


### Corner Cases

In some languages like chinese, there can be multiple "words" extracted that are considered to be at the same position in the text. For example 计算所 gives 计算 and 计算所, that are at the same position in the text but don't have the same length, the tokenizer should support that behavior.


## Future possibilities


- The field `char_index` is usefull for the current meilisearch but not in the future core-engine, It would be deprecated in future versions
- We should add a way to configure tokenizer forcing a specific language/script
- We should add a way to configure tokenizer adding custom stopword
- We should add a way to configure tokenizer activating/deactivating stopword detection
- We should add a way to configure tokenizer whitelisting/backlisting separators
- The tokenizer specified here is based on scripts, we should base it on languages to be able to have default stop-words for each language
