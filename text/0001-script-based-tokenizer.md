- Title: Script Based Tokenizer
- Start Date: 2020-10-27
- specification PR: meilisearch/specifications#2
- Meilisearch Issue: meilisearch/Meilsearch#624

## Feature Description and Interaction

### Summary

The first step of document indexing in the Meilisearch engine is tokenization. Tokenization is the action of taking a sentence and splitting it in units of language called tokens. The tokenization task is highly language dependant and is a critical factor in the quality of the search results.

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
  > language detection is low but the script is acceptable.
  > note: Sonic also uses whatlang to perform the tokenization, it could be interesting to check out how they do it.
- > **toku (@qdequele):**
  > in a R&D project, @qdequele was able to detect language based on stop word distribution in a text.
  > If, in a latin script, there is lot of French stop words then the text language is probably french.

**other solution that advertise multilingual support:**
- Sonic uses whatlang to perform the tokenization, it could be interesting to checkout how they do it: https://github.com/valeriansaliou/sonic/tree/master/src/lexer
	- Sonic uses whatlang to detect the languages but doesn't actually seem to use it to segment the text. It simply uses unicode segmentation, I can't really explain what they actually do with the language information.
- tantivy advertise good multilingual support: https://github.com/tantivy-search/tantivy/tree/main/src/tokenizer
	- Tantivy is similar to elastic in the sense that you can set up a custom text analyzer. The difference is that it is only made of `tokenizer` -> `token_filter`. Tantivy also provides a collection of tokenizer to choose from. Tokens are rather simple and do not contain any metadata, except for their position.
- How elastic search handle it: https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html:
	- Elastic proposed to create custom text analyzer. A text analyzer is a pipelined text processor with the following components: `char_filter` -> `tokenizer` -> `token_filter`. There are multiple different tokenizers that can be chosen, depending on the use-case. This is a bit complicated, but I also think that advanced users should be able to choose the tokenizer they want. (default behavior is us guessing what's the best tokenizer to use.)
- Algolia:
    - Algolia uses multiple techniques to handle tokenization. They start by [normalizing](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/in-depth/normalization/) the text (lowercase, unidecode, transform traditional Chinese to modern, etc). Then, they tokenize the normalized data. It seems that the tokenization is based on two techniques. The first one is by [defining separators](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/in-depth/tokenization/) (space, comma, carriage return, etc) and the other one is with [dictionaries](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/#using-dictionaries). The language is not automatically detected, [it must be set by the user](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/handling-natural-languages-nlp/#no-automatic-language-detection). They also have dictionaries for plurals. 

### Explanation

In order to use the tokenizer, all the user has to do is to instantiate a `Tokenizer`, call `tokenize(&str)` on it and iterate over the emitted tokens:

```rust
let stop_words = Set::from_iter(["the", "of"].iter()).unwrap();
let analyzer = Analyzer::new(AnalyzerConfig::default_with_stopwords(&stop_words));

let orig = "The quick (\"brown\") fox can't jump 32.3 feet, right? Brr, it's 29.3°F!";
let analyzed = analyzer.analyze(orig);
let mut analyzed = analyzed.tokens();
assert_eq!("the", analyzed.next().unwrap().text());
assert_eq!(" ", analyzed.next().unwrap().text());
assert_eq!("quick", analyzed.next().unwrap().text());
assert_eq!(" ", analyzed.next().unwrap().text());
assert_eq!("(", analyzed.next().unwrap().text());
assert_eq!("\"", analyzed.next().unwrap().text());
assert_eq!("brown", analyzed.next().unwrap().text());
```

The call to the tokenize method allows the reuse of the same `Tokenizer` instance, and keep its configuration state and allocations.

Below are examples of the integration of the new tokenizer in existing code:

- Highlight in @kerollmops milli:
```rust
// new tokenizer
fn highlight_record(record: &mut IndexMap<String, String>, words: &HashSet<String>) {
   let stop_words = Set::default();
    let analyzer = Analyzer::new(AnalyzerConfig::default_with_stopwords(&stop_words));

    for (_key, value) in record.iter_mut() {
        let old_value = mem::take(value);
        let analyzed = analyzer.analyze(&old_value);
        
        for (original, token) in analyzed.reconstruct() {
            if token.is_word() {
                let to_highlight = words.contains(&token.text());
                if to_highlight { value.push_str("<mark>") }
                value.push_str(original);
                if to_highlight { value.push_str("</mark>") }
            } else {
                value.push_str(original);
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

As we can see, the changes that need to be made are very minimal: this is because efforts have been made to make its API close to the previous one.

### Impact on documentation

This feature should not impact meilisearch users' documentation. 
In future versions, we will probably provide a way to configure tokenizer and this will be discussed in a new specification.

## Technical Specifications

### Architecture

The new version of the tokenizer will replace the current version as a standalone library.

![Tokenizer](https://user-images.githubusercontent.com/6482087/102896344-8560d200-4466-11eb-8cfe-b4ae8741093b.jpg)
### Implementation Details

We want to support different tokenizers based on the language of the text that needs to be indexed. For this, we may need to change the tokenizer we are using while indexing, depending on the language and the script, detected by `whatlang`. The Analyzer provides an interface that abstracts this need away from the consumer of the tokens.

#### Pipeline

A pipeline is a structure that contains all the steps needed to tokenize a specific language or script:
- One or several preprocessors to prepare text before tokenization e.g. traditional Chinese translation or erasing some characters;
- A tokenizer to transform the text into an iterator of tokens;
- One or several normalizers to normalize tokens for indexation e.g. deunicode or lowercase.
```rust
pub struct Pipeline {
    pre_processor: Box<dyn PreProcessor + 'static>,
    tokenizer: Box<dyn Tokenizer + 'static>,
    normalizer: Box<dyn Normalizer + 'static>,
}
```

#### Configuration

The tokenizer is configured at initialization. A `Default` implementation is provided with the settings we believe suits most cases.
The configuration options, for now, are:
- the `stop_words`
- the `pipeline_map` that links a `(Script, Language)` to a `Pipeline`. This defaults to the mapping we find the best fitting for most cases. These settings should be left as is now, but will allow the user to define Pipelines in the future.

```rust
pub struct AnalyzerConfig<'a, A> {
    /// language specialized pipeline, this can be switched during
    /// document tokenization if the document contains several languages
    pub pipeline_map: HashMap<(Script, Language), Pipeline>,
    pub stop_words: &'a Set<A>,
}
```

#### Analyzer

The analyzer exposes an abstracted interface to the tokenization process. Its API is standard:

```rust
use crate::token::Token;
use crate::tokenizer::Tokenizer;

pub struct Analyzer<'a, A> {
    config: AnalyzerConfig<'a, A>,
}

impl<'a, A> Analyzer<'a, A>
where
    A: AsRef<[u8]>
{
    /// create a new tokenizer detecting script
    /// and chose the specialized internal tokenizer
    pub fn new(config: AnalyzerConfig<'a, A>) -> Self { unimplemented!() }

    /// Builds an `AnalyzedText` instance with the correct analyzer pipeline, and pre-processes the
    /// text.
    ///
    /// If an analysis pipeline exists for the inferred `(Script, Language)`, the analyzer will look
    /// for a user specified default `(Script::Other, Language::Other)`. If the user default is not
    /// specified, it will fallback to `(IdentityPreProcessor, UnicodeSegmenter, IdentityNormalizer)`.
    ///
    /// ```rust
    /// use meilisearch_tokenizer::{Analyzer, AnalyzerConfig};
    /// use fst::Set;
    /// // defaults to unicode segmenter with identity preprocessor and normalizer.
    /// let stop_words = Set::from_iter([""].iter()).unwrap();
    /// let analyzer = Analyzer::new(AnalyzerConfig::default_with_stopwords(&stop_words));
    /// let analyzed = analyzer.analyze("The quick (\"brown\") fox can't jump 32.3 feet, right? Brr, it's 29.3°F!");
    /// let mut tokens = analyzed.tokens();
    /// assert!("the" == tokens.next().unwrap().text());
    /// ```
    pub fn analyze<'t>(&'t self, text: &'t str) -> AnalyzedText<'t, A> { unimplemented!() }
}

/// result of analyzer.analyze() function, made to choose how tokens will be returned.
pub struct AnalyzedText<'a, A> {}

impl<'a, A> AnalyzedText<'a, A>
where
    A: AsRef<[u8]>
{
    /// Returns a `TokenStream` for the Analyzed text.
    pub fn tokens(&'a self) -> TokenStream<'a> { unimplemented!() }

    /// Attaches each token to its corresponding portion of the original text.
    pub fn reconstruct(&'a self) -> impl Iterator<Item = (&'a str, Token<'a>)>  { unimplemented!() }
}
```

#### Token

The `Token` is is the result of an iteration of the `Analyzer`,
it wraps a `&str`, a byte slice containing one or several characters, with some informations about:
- the `kind: TokenKind` defining if the token is a `Word`, a `Separator` or a `StopWord`
- the `char_index: usize` defining the index of the first character of the `Token` in the whole original text
- the `byte_start/byte_end: usize` defining the indexes of start and end of the byte slice in the whole original text

:warning: The wrapped byte slice should not be considered as a subset of the original text 
but as a normalized version of the subset.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum SeparatorKind {
    Hard,
    Soft,
}

#[derive(Debug, Clone, Copy, PartialEq)]
pub enum TokenKind {
    Word,
    /// the token is a stop word,
    /// meaning that it can be ignored to optimize size and performance or be indexed as a Word
    StopWord,
    /// the token is a separator,
    /// meaning that it shouldn't be indexed but used to determine word proximity
    Separator(SeparatorKind),
    Unknown,
}
```

The emitted token position is relative to the original text, this information is reliable. The `word` is normalized if need be.

```rust
#[derive(Debug, Clone, Default)]
pub struct Token<'a> {
    pub kind: TokenKind,
    pub word: Cow<'a, str>,
    /// index of the first character of the word
    pub char_index: usize,
    /// indexes of start and end of the byte slice
    pub byte_start: usize,
    pub byte_end: usize,
}

impl<'a> Token<'a> {
    // return the normalized version of the token
    pub fn text(&self) -> &str { unimplemented!() }

    // return the size in byte of the original text
    pub fn byte_len(&self) -> usize { unimplemented!() }

    // return the TokenKind of the token
    pub fn kind(&self) -> TokenKind { unimplemented!() }

    // return true if the TokenKind of the token is TokenKind::Word
    pub fn is_word(&self) -> bool { unimplemented!() }

    // return Some(SeparatorKind) if the TokenKind of the token is TokenKind::Separator(SeparatorKind),
    // None if the token is not a separator
    pub fn is_separator(&self) -> Option<SeparatorKind> { unimplemented!() }

    // return true if the TokenKind of the token is TokenKind::StopWord
    pub fn is_stopword(&self) -> bool { unimplemented!() }
}
```

#### Internal Tokenizer trait

The `InternalTokenizer` trait provides a common interface to adapt other tokenizers to the tokenizer. This allows the extensibility of the current tokenizer to other languages.

```rust
/// iterator over tokens processed by the specialized tokenizer
pub struct TokenStream<'a> {
    pub(crate) inner: Box<dyn Iterator<Item = Token<'a>> + 'a>
}

impl<'a> Iterator for TokenStream<'a> {
    type Item = Token<'a>;

    fn next(&mut self) -> Option<Self::Item> {
        self.inner.next()
    }
}

/// trait defining an internal tokenizer,
/// an internal tokenizer should be a script specialized tokenizer,
/// this should be implemented as an `Iterator` with `Token` as `Item`,
pub trait Tokenizer: Sync + Send {
    /// create the tokenizer based on the given `text` and `char_index`
    fn tokenize<'a>(&self, s: &'a ProcessedText<'a>) -> TokenStream<'a>;
}
```
- use of `jieba` for Chinese tokenization
- use of `unicode-segmenter` for other segmentations

### Corner Cases

In some languages like Chinese, there can be multiple "words" extracted that are considered to be at the same position in the text. For example, 计算所 gives 计算 and 计算所, that are at the same position in the text but don't have the same length, the tokenizer should support that behavior. See #5.
> return several token as the same word_position seems to be over-engineering, we may want to have only 1 token by iteration making API simplier
## Future possibilities

- We should add a way to configure the tokenizer to enforce a specific language/script
- We should add a way to configure tokenizer whitelisting/blacklisting separators
- The tokenizer specified here is based on scripts, we should base it on languages to be able to have default stop-words for each language
- The chinese tokenizer is a complicated subject. The first implementation will simply adapt jieba's `cut` method. In another [specification](https://github.com/meilisearch/specifications/pull/5), we'll think about improving this, and this will probably require the help of a native mandarin speaker input.
- We will want in the future to allow user configuration for the tokenizer. This is taken into account in the design of the new Tokenizer.
- Normalized synonyms (#964)
