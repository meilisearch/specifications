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
    use fst::Set;
    
    use charabia::TokenizerBuilder;
    
    // text to tokenize.
    let orig = "The quick (\"brown\") fox can't jump 32.3 feet, right? Brr, it's 29.3°F!";
    
    // create the builder.
    let mut builder = TokenizerBuilder::new();
    
    // create a set of stop words.
    let stop_words = Set::from_iter(["the"].iter()).unwrap();
    
    // configurate stop words.
    builder.stop_words(&stop_words);
    
    // build the tokenizer passing the text to tokenize.
    let tokenizer = builder.build();
    
    // tokenize original string
    let mut tokens = tokenizer.tokenize(orig);
    
    let Token { lemma, kind, .. } = tokens.next().unwrap();
    assert_eq!(lemma, "the");
    assert_eq!(kind, TokenKind::Word);

    let Token { lemma, kind, .. } = tokens.next().unwrap();
    assert_eq!(lemma, " ");
    assert_eq!(kind, TokenKind::Separator(SeparatorKind::Soft));

    let Token { lemma, kind, .. } = tokens.next().unwrap();
    assert_eq!(lemma, "quick");
    assert_eq!(kind, TokenKind::Word);
```

The call to the tokenize method allows the reuse of the same `Tokenizer` instance, and keep its configuration state and allocations.

Below are examples of the integration of the new tokenizer in existing code:

- Highlight in @kerollmops milli:
```rust
// new tokenizer
fn highlight_record(record: &mut IndexMap<String, String>, words: &HashSet<String>) {
    // create the builder.
    let mut builder = TokenizerBuilder::new();
    
    // create a set of stop words.
    let stop_words = Set::from_iter(["the"].iter()).unwrap();
    
    // configurate stop words.
    builder.stop_words(&stop_words);
    
    // build the tokenizer passing the text to tokenize.
    let tokenizer = builder.build();

    for (_key, value) in record.iter_mut() {
        let old_value = mem::take(value);
        // reuse tokenizer at each iteration
        let tokens = tokenizer.reconstruct(&old_value);
        
        for (original, token) in tokens {
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

The new version of the tokenizer will replace the current version as a [standalone library named charabia](https://crates.io/crates/charabia).

### Implementation Details

We want to support different tokenizers based on the language of the text that needs to be indexed. For this, we may need to change the tokenizer we are using while indexing, depending on the language and the script, detected by `whatlang`. The Tokenizer provides an interface that abstracts this need away from the consumer of the tokens.

See [the official documentation](https://docs.rs/charabia) to know more about the API of the library.

See the repository [Contributing.md](https://github.com/meilisearch/charabia/blob/main/CONTRIBUTING.md) to know more about contribution that can be made by the community.


## Future possibilities

- We should add a way to configure the tokenizer to enforce a specific language/script
- We should add a way to configure tokenizer whitelisting/blacklisting separators
- The tokenizer specified here is based on scripts, we should base it on languages to be able to have default stop-words for each language
- We will want in the future to allow user configuration for the tokenizer. This is taken into account in the design of the new Tokenizer.
