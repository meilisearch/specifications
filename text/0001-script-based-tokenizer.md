
- Title: Script Based Tokenizer
- Start Date: 2020-10-27
- specification PR: meilisearch/specifications#1
- Meilisearch Issue: meilisearch/Meilsearch#624

# Summary
[summary]: #summary

Tokenizer's principal role is to split documents into words (tokens) so each document can be indexed by it's contained words. It is also needed to split a search a query into token, and search on the work index.

# Motivation
[motivation]: #motivation

It turns out that splitting texts into tokens is a complicated task, especially when you need to handle more than a single language. While the task for english may seem trivial at first glance (mostly split white space), Chinese tokenization, for example is completely different (no spaces between words, groups of 1, 2 or 3 characters...).

Actual Tokenizer is naive and partially handle Latin Scripts only, lot of users ask for a smarter tokenizer to handle their own languages.
We should implement a new tokenizer which detect script and tokenize based on it.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

TBD

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Tokenizer
The tokenizer serves s a proxy for other tokenizers, specialized in the language detected by the Tokenizer. It is instantiated with a String, and is then polled for tokens until it's delepted:
```rust
let tokenizer = Tokenizer::new("The quick brown fox jumps over the lazy dog");
assert_eq!(tokenizer.next().txt(), "The")
\```
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

## Token

```rust
/// script of a token (https://docs.rs/whatlang/0.10.0/whatlang/enum.Script.html)
pub type Script = whatlang::Script;

/// atomic item returned by `Tokenizer::next()`,
/// it determine the type of the token and contains a `WordSlice`
pub enum Token<'a> {
    /// the token is a word,
    /// meaning that it should be indexed as an important part of the document
    Word(WordSlice<'a>),
    /// the token is a stop word,
    /// meaning that it can be ignored to optimize size and performance or be indexed as a Word
    StopWord(WordSlice<'a>),
    /// the token is a separator,
    /// meaning that it shouln't be indexed but used to determine word proximity
    Separator(WordSlice<'a>)
}
impl<'a> Token<'a> {
    fn txt(&self) -> &str { unimplemented!() }
}
/// The script, the char_index and the content of the token
pub struct WordSlice<'a> {
    /// content of the token
    pub word: &'a str,
    /// index of the first character of the token in the whole document
    pub char_index: usize,
}
```

## Interal Tokenizer trait

```rust
use crate::token::{Token, WordSlice, Script};

/// trait defining an internal tokenizer,
/// an internal tokenizer should be a script specialized tokenizer,
/// this should be implemented as an `Iterator` with `Token` as `Item`,
pub(crate) trait InternalTokenizer<'a>: Iterator<Item = Token<'a>> {
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


# Drawbacks
[drawbacks]: #drawbacks

TBD

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

TBD
<!-- - Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this? -->

# Prior art
[prior-art]: #prior-art

- [unicode-segmentation](https://github.com/unicode-rs/unicode-segmentation) for Latin scripts
- [Jieba](https://github.com/messense/jieba-rs) for Chinese
- [Lindera](https://github.com/lindera-morphology/lindera) for Japanese and Korean

<!-- Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that rust sometimes intentionally diverges from common language features. -->

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- Is `Token` sufficiently exhaustive?
<!-- - What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC? -->

# Does it Impact Meilisearch Documentation? If yes, how much and which part?

It should not impact the meilisearch documentation.

# Future possibilities
[future-possibilities]: #future-possibilities

TBD

<!-- Think about what the natural extension and evolution of your proposal would
be and how it would affect the language and project as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the project and language in your proposal.
Also consider how the this all fits into the roadmap for the project
and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information. -->
