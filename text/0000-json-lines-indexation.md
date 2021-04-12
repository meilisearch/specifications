- Title: Json Lines Indexation
- Start Date: 2021-04-12
- Specification PR: TBD
- MeiliSearch Issues: [#128](https://github.com/meilisearch/transplant/issues/128),[#1332](https://github.com/meilisearch/MeiliSearch/issues/1332)

## Feature Description and Interaction

### Summary

The initiation step of document indexing is to send some file matching a format to be parsed and tokenized in order to give search results to end-users. A [JSON Lines](https://jsonlines.org/) is easier to use than a CSV file because it propose a convenient format for storing structured data.

### Motivation

We want to provide our users with an always improved usage experience. Currently, the engine only accepts JSON format as a data source. We want to give users the possibility of another simple file format to use. Thus, give them more versatility at the data source choices for the indexation step.

Writing performance is also considered as a motivation since JSON Lines parsing is less CPU and memory intensive than parsing JSON because every new lines represent a separate entry, making the JSON Lines file streamable. Thus, more suited for indexing a consequent data set.

### Explanation
### Impact on documentation
### Impact on SDKs

## Technical Specifications

### Architecture
### Technical Limitations
### Corner Cases

## Future possibilities
