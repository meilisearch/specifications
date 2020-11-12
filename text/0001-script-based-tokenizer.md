- Title: Script Based Tokenizer
- Start Date: 2020-10-27
- specification PR: meilisearch/specifications#2
- Meilisearch Issue: meilisearch/Meilsearch#624

## Feature Description and Interaction

### Summary

The first step of document indexing in the Meilisearch engine is tokenization. Tokenization in the action of taking a sentence and splitting in units of language called tokens. The tokenization task is highly language dependant and is a critical factor in the quality of the search results.

### Motivation

We want to provide our users with an always improved searching experience. For that matter, it is critical for us to improve the performance of our tokenizer, and to provide better support for multilingual tokenization.

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
	- Elastic proposed to create custom text analyzer. A text analyzer is a pipelined text processor with the following components: `char_filter` -> `tokenizer` -> `token_filter`. There are multiple diffferent tokenizer that can be chosed, depending on the usecase. This is a bit complicated, but I also think that advanced users should be able to chose the tokenizer they want. (default behaviour is us guessing what's the best tokenizer to use.)
- Algolia:
    - Algolia use multiple techniques to handle tokenization. They start by [normalize](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/in-depth/normalization/) the data (lowercase, unidecode, transform traditional Chinese to modern, etc). Then, they tokenize the normalized data. It seems that the tokenization is based on two techniques. The first one is by [defining separators](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/in-depth/tokenization/) (space, coma, carriage return, etc) and the other one is with [dictionaries](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/#using-dictionaries). The language is not automaticaly detected, [it must be set by the user](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/#no-automatic-language-detection). They also have dictionaries for plurials. 

### Explanation

In order to use the tokenizer, all the user has to do is to instantiate a `Tokenizer`, call `tokenize(&str)` on it and iterate over the emitted tokens:

```rust
let config = TokenizerConfig::default().set_stopwords(&["the", "of"]);
let tokenizer = Tokenizer::new(config);
let groups = tokenizer.tokenize("The quick brown fox jumps over the lazy dog");
let token_group = groups.next();
assert_eq!(token_group.tokens().map(|t| t.text()).collect(), ["The"]);
assert_eq!(token_group.normalized().map(|t| t.text()).collect(), ["the"]);
let token_group = tokens.next();
assert_eq!(token_group.tokens().map(|t| t.text()).collect(), [" "])
let token_group = groups.next();
assert_eq!(token_group.tokens().map(|t| t.text()).collect(), ["quick"])
```

The call to the tokenize method allows the reuse of the same `Tokenizer` instance, and keep it's configuration state and allocations.

Bellow are example of the integration of the new tokenizer in existing code:

- Highlight in @kerollmops milli:
```rust
// new tokenizer
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

- Indexing in @kerollmops milli

```rust
// new tokenizer
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

- index token method in the current Meilisearch engine

```rust
// new tokenizer, missing details
fn index_token<A>(
    group: TokenGroup,
    id: DocumentId,
    indexed_pos: IndexedPos,
    word_limit: usize,
    stop_words: &fst::Set<A>,
    words_doc_indexes: &mut BTreeMap<Word, Vec<DocIndex>>,
    docs_words: &mut HashMap<DocumentId, Vec<Word>>,
) -> bool
where A: AsRef<[u8]>,
{
    // we get normalized token: token normalization is performed by the tokenizer
    for token in group.normalized() {
        if token.token_index() >= word_limit {
            return false;
        }

        if !token.is_stopword() {
            match token_to_docindex(id, indexed_pos, token) {
                Some(docindex) => {
                    let word = Vec::from(token.word);

                    if word.len() <= WORD_LENGTH_LIMIT {
                        words_doc_indexes
                            .entry(word.clone())
                            .or_insert_with(Vec::new)
                            .push(docindex);
                        docs_words.entry(id).or_insert_with(Vec::new).push(word);
                    }
		    // no need to reindex a normalized version, the query string will be normalized too
                }
                None => return false,
            }
        }
    }

    true
}
```
```rust
// original
fn index_token<A>(
    token: Token,
    id: DocumentId,
    indexed_pos: IndexedPos,
    word_limit: usize,
    stop_words: &fst::Set<A>,
    words_doc_indexes: &mut BTreeMap<Word, Vec<DocIndex>>,
    docs_words: &mut HashMap<DocumentId, Vec<Word>>,
) -> bool
where A: AsRef<[u8]>,
{
    if token.index >= word_limit {
        return false;
    }

    // This is an instance of a NormalizedToken
    let normalized = token.text();
    let token = Token {
        word: &lower,
        ..token
    };

    if !stop_words.contains(&token.word) {
        match token_to_docindex(id, indexed_pos, token) {
            Some(docindex) => {
                let word = Vec::from(token.word);

                if word.len() <= WORD_LENGTH_LIMIT {
                    words_doc_indexes
                        .entry(word.clone())
                        .or_insert_with(Vec::new)
                        .push(docindex);
                    docs_words.entry(id).or_insert_with(Vec::new).push(word);

                    if !lower.contains(is_cjk) {
                        let unidecoded = deunicode_with_tofu(&lower, "");
                        if unidecoded != lower && !unidecoded.is_empty() {
                            let word = Vec::from(unidecoded);
                            if word.len() <= WORD_LENGTH_LIMIT {
                                words_doc_indexes
                                    .entry(word.clone())
                                    .or_insert_with(Vec::new)
                                    .push(docindex);
                                docs_words.entry(id).or_insert_with(Vec::new).push(word);
                            }
                        }
                    }
                }
            }
            None => return false,
        }
    }

    true
}
```

As we can see, the changes that need to be made are very minimal: this is because efforts have been made to make it's API close to the previous one.

### Impact on documentation

This feature should not impact the meilisearch user documentation,
in future versions we will probably provide a way to configure tokenizer and this will be discussed in a new specification.

## Technical Specifications

### Architecture

The new version of the tokenizer will replace the current version as a standalone library.

We want to support different tokenizers based on the language of the text to be indexed. For this we may need to change the tokenizer we are using while indexing, depending on the language and the script. The tokenizer provides an interface that abstract this need away from the consumer of the tokens.

#### Configuration

The tokenizer in configured at initialization. A `Default` implementation is provided with the settings we believe suits most cases.
The configuration options, for now, are:
- the `stopwords`, default to empty
- the `tokenizers_map` that links a `(Script, Language)` to a `Box<dyn InternalTokenizer>`. This defaults to the mapping we find best fitting for most cases. This settings should be left as is now, but will allow user defined tokenizers in the future.

```rust
pub struct TokenizerConfig {
	stopwords: Vec<String>,
	tokenizer_map: HashMap<(Script, Language), Box<dyn InternalTokenizer>>,
}
```

_edit 1: return several token as the same word_position seems to be overenginering, we may want to have only 1 token by iteration making API simplier_

_edit 2: we have to normalize text in order to have a more accurate tokenization (traditional vs simplified chinese)_

#### Tokenizer

The tokenizer exposes an abstracted interface to the tokenization process. Its API is standard:

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
    tokenizer_map: HashMap<(Script, Language), Box<dyn InternalTokenizer>>,
}

impl<'a> Tokenizer<'a> {
    /// create a new tokenizer detecting script
    /// and chose the specialized internal tokenizer
    fn new(config: TokenizerConfig) -> Self { unimplemented!() }
    // Analyses the text (typically a field) and select the correct tokenizer from the tokenizer_map, return an iterator of token groups, from the Cow<[Token<'a'>]> emitted from the internal tokenizer
    fn tokenize(s: &'a str) -> impl Iterator<Item = &'a Token<'a>>
}
```

#### Token

```rust
enum TokenKind {
    Word,
    /// the token is a stop word,
    /// meaning that it can be ignored to optimize size and performance or be indexed as a Word
    StopWord,
    /// the token is a separator,
    /// meaning that it shouldn't be indexed but used to determine word proximity
    Separator
}
```

```rust
pub struct Token<'a> {
    kind: TokenKind,
    word: Cow<str>,
    /// index of the first character of the word
    char_start: usize,
    char_end: usize,
    /// byte index of the first character of the word
    byte_index: usize, // usefull?
    /// position of the token in the token stream
    token_index: usize, // useful?
}

mpl Token {
    fn token_len(&self) -> usize {}
    fn kind(&self) -> TokenKind {}
    fn is_word(&self) -> bool {}
    fn is_separator(&self) -> bool {}
    fn is_stopword(&self) -> bool {}
}
```

#### Internal Tokenizer trait

The `InternalTokenizer` traits provides a common interface to adapt other tokenizers to the tokenizer. This allows extensibility of the current tokenizer to other languages.

```rust
use crate::token::{Token, WordSlice, Script};

/// trait defining an internal tokenizer,
/// an internal tokenizer should be a script specialized tokenizer,
/// this should be implemented as an `Iterator` with `Token` as `Item`,
pub(crate) trait InternalTokenizer<'a> {
    /// the tokenize method takes text as an input and emits tokens
    /// Tokens must be returned with an monotonically increasing token position, and tokens with the same position must be grouped together.
    fn tokenize(text: &'a str) -> impl Iterator<Item = Token<'a>>>;
}
```


### Implementation Details

- use of jieba for chinese tokenization
- use of unicode-segmenter for other segmentations
- Use StreamingIterator instead of Iterator

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
