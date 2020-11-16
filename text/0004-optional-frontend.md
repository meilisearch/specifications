- Title: Optional Meilisearch Front-end
- Start Date: 2020/11/16
- specification PR:
- Meilisearch Issue: #411

# Optional Meilisearch Front-end

## First section: Feature Description and Interaction

### Summary

For testing purposes, MeiliSearch is shipped with a frontend that is served on the server's root. This provides the user with a basic GUI to perform searches on his document collection. While this is useful in a development environment, this is not necessarily desirable in a production environment. This specification propose to make this frontend optional thanks to binary setting.

### Motivation

In most production cases, there is no need for this frontend, and exposing it is a bad practice. We want to provide an opt-out option for this feature.

### Prior Art and R&D

N.A

### Explanation

On MeiliSearch startup, we would like to pass a flag `--disable-web-gui` that disables the web user interface.

### Impact on documentation

This new feature adds a new flag to the binary, that needs to be documented [here](https://docs.meilisearch.com/guides/advanced_guides/configuration.html).

## Second Section: Technical Specifications

### Architecture

N.A

### Implementation Details

A flag needs to be declared in the [`Opt` struct](https://github.com/meilisearch/MeiliSearch/blob/master/meilisearch-http/src/option.rs#L16), then, these [two lines](https://github.com/meilisearch/MeiliSearch/blob/master/meilisearch-http/src/lib.rs#L49-L50) need to be disabled.

### Corner Cases

TDB

## Third Section: Future possibilities
N.A
