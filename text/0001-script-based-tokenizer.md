- Title: Script Based Tokenizer
- Start Date: 2020-10-27
- specification PR: meilisearch/specifications#2
- Meilisearch Issue: meilisearch/Meilsearch#624

## Feature Description and Interaction

### Summary

Tokenizer's principal role is to split documents into words (tokens) so each document can be indexed by it's contained words. It is also needed to split a search query into tokens, and search in the words index.

### Motivation

It turns out that splitting texts into tokens is a complicated task, especially when you need to handle more than a single language. While the task for english may seem trivial at first glance (mostly split white space), Chinese tokenization, for example is completely different (no spaces between words, groups of 1, 2 or 3 characters...).

Actual Tokenizer is naive and partially handle Latin Scripts only, lot of users ask for a smarter tokenizer to handle their own languages.
We should implement a new tokenizer which detect script and tokenize based on it.

### Prior Art and R&D

**tokenization:**
- > **[unicode-segmentation](https://github.com/unicode-rs/unicode-segmentation):**
  > tokenizer which follow the [Standard Annex #29: Unicode Text Segmentation](http://www.unicode.org/reports/tr29/),
  > this tokenizer seems promising for Latin scripts.
- > **[Jieba](https://github.com/messense/jieba-rs):**
  > tokenizer specialized in Chinese languages
- > **[Lindera](https://github.com/lindera-morphology/lindera):**
  > Japanese and Korean

**lang/script detection:**
- > **[whatlang](https://github.com/greyblake/whatlang-rs):**
  > whatlang is able to detect script or/and language in a text,
  > language detection is low but script is acceptable.
  > note: Sonic also uses whatlang to peform the tokenization, it could be interesting to checkout how they do it.
- > **toku (@qdequele):**
  > in a R&D project, @qdequele was able to detect language based on stop word distribution in a text.
  > If, in a latin script, there is lot of French stop words then the text language is probably french.

**other solution that advertise multilingual support:**
- Sonic uses whatlang to peform the tokenization, it could be interesting to checkout how they do it: https://github.com/valeriansaliou/sonic/tree/master/src/lexer
	- Sonic uses whatlang to detect the languages, but don't acutally seems to use it to segment the text. It simply uses unicode segmentation, I can't really explain what they actually do with the language information.
- tantivy advertise good multilingual support: https://github.com/tantivy-search/tantivy/tree/main/src/tokenizer
	- Tantivy is similar to elastic in the sense that you can setup a custom text analyzer. The difference is that it is only made of `tokenizer` -> `token_filter`. Tantivy also provides a colleciton of tokenizer to choose from. Token are rather simple and do not contain any metadata, except for it's position.
- How elastic search handle it: https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html:
	- Algolia proposed to create custom text analyzer. A text analyzer is a pipelined text processor with the collowing components: `char_filter` -> `tokenizer` -> `token_filter`. There are multiple diffferent tokenizer that can be chosed, depending on the usecase. This is a bit complicated, but I also think that advanced users should be able to chose the tokenizer they want. (default behaviour is us guessing what's the best tokenizer to use.)
- Algolia:
	- Algolia seems to use default word segmentation use user configurable separators. The use can also set the `queryLanguage` or provide a list of `stopWords`, but it is unclear how this work under the hood for languages such as Mandarin. They also word decompounging, meaning that they can split words that are mode of multiple words (mostly germanic/nordic languages apparently).

### Explanation

The tokenizer serves as a proxy for other tokenizers, specialized in the language detected by the Tokenizer. It is instantiated with a String, and is then polled for tokens until it's delepted:

```rust
let tokenizer = Tokenizer::new("The quick brown fox jumps over the lazy dog");
let token_group = tokenizer.next();
assert_eq!(token_group.tokens().map(|t| t.text()).collect(), ["The"]);
assert_eq!(token_group.normalized().map(|t| t.text()).collect(), ["the"]);
let token_group = tokenizer.next();
assert_eq!(token_group.tokens().map(|t| t.text()).collect(), [" "])
let token_group = tokenizer.next();
assert_eq!(token_group.tokens().map(|t| t.text()).collect(), ["quick"])
```

#### highlight

```rust
fn highlight_record(record: &mut IndexMap<String, String>, words: &HashSet<String>) {
  for (_key, value) in record.iter_mut() {
    let old_value = mem::take(value);
    let tokenizer = Tokenizer::new(&the_string);
    for token_group in tokenizer {
      // get the longest normalized token
      let token = token_goup.normalized().max_by(|a, b| a.token_len().cmp(b.token_len()));
      if token.kind() == TokenKind::Word {
        let normalized_token = token.text();
        let to_highlight = words.contains(&normalized_token);
        if to_highlight { value.push_str("<mark>") }
        value.push_str(token.original.text());
        if to_highlight { value.push_str("</mark>") }
      } else {
        value.push_str(token);
      }
    }
  }
}
```

```rust
// original
fn highlight_record(record: &mut IndexMap<String, String>, words: &HashSet<String>) {
    for (_key, value) in record.iter_mut() {
        let old_value = mem::take(value);
        for (token_type, token) in simple_tokenizer(&old_value) {
            if token_type == TokenType::Word {
                let lowercase_token = token.to_lowercase();
                let to_highlight = words.contains(&lowercase_token);
                if to_highlight { value.push_str("<mark>") }
                value.push_str(token);
                if to_highlight { value.push_str("</mark>") }
            } else {
                value.push_str(token);
            }
        }
    }
}
```

#### Store

```rust
let tokenizer = Tokenizer::new(&content);
for (pos, token_group) in tokenizer.filter_map(only_token).enumerate().take(MAX_POSITION) {
    let position = (attr as usize * MAX_POSITION + pos) as u32;
    for token in token_group.normalized() {
        words_positions.entry(token.txt()).or_insert_with(SmallVec32::new).push(position);
    }
}
```

```rust
// original
for (pos, token) in simple_tokenizer(&content).filter_map(only_token).enumerate().take(MAX_POSITION) {
    let word = token.to_lowercase();
    let position = (attr as usize * MAX_POSITION + pos) as u32;
    words_positions.entry(word).or_insert_with(SmallVec32::new).push(position);
}
```

### Impact on documentation

This feature should not impact the meilisearch user documentation,
in future versions we will probably provide a way to configure tokenizer and this will be discussed in a new specification.

## Technical Specifications

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
pub struct TokenGroup<'a> {
    tokens: Cow<'a, '[Token<'a>]>,
    script: Script,
}

struct NormalizedTokenIter {
    script: Script,
    token: Iter<Token<'a>>
}

impl Deref for NormalizedToken {

}

struct NormalizedToken<'a> {
    original: Token<'a>,
    normalized: Cow<str>,
}

impl NormalizedToken {
    fn text() -> &str {}
}

impl Iterator for NormalizedTokenIter {
    type Item = NormalizedToken;
    
    fn next() -> Self::Item {
        // token.word = self.normalize(token)
    }
}

impl<'a> TokenGroup<'a> {
    fn tokens(&self) -> impl Iterator<Item = Token<'a>> {}
    fn normalized() -> NormalizedTokenIterator {}
}

enum TokenKind {
    Word,
    /// the token is a stop word,
    /// meaning that it can be ignored to optimize size and performance or be indexed as a Word
    StopWord,
    /// the token is a separator,
    /// meaning that it shouldn't be indexed but used to determine word proximity
    Separator
}

/// Determine the type of the token and contains a `WordSlice`
pub struct Token<'a> {
    kind: TokenKind,
    word: &'a str,
    /// index of the first character of the token in the whole document
    char_index: usize,
    byte_index: usize,
    token_index: usize,
}

impl Token {
    fn token_len(&self) -> usize {}
    fn kind(&self) -> TokenKind {}
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
- The chinese tokenizer is a complicated subject. The first implementation will simply adapt jieba's `cut` method. In another [specification](https://github.com/meilisearch/specifications/pull/5), we'll think about improving this, and this will probably require the help of a native mandarin speaker input.
- We will want in the future to allow user configuration for the tokenizer. This is taken into account in the design of the new Tokenizer.
