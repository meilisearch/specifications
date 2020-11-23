- Title: Optional Meilisearch Front-end
- Start Date: 2020/11/16
- specification PR: #8
- Meilisearch Issue: #411

# Optional Meilisearch Front-end

## First section: Feature Description and Interaction

### Summary

For testing purposes, MeiliSearch is shipped with a frontend that is served at the server's root. This provides the user with a basic GUI to perform searches on his documents. While this is useful in a development environment, this is not necessarily desirable in a production environment. This specification proposes to remove this frontend in a production environment.

### Motivation

This frontend was developed for testing purposes, therefore it should only be present in a development environment.

### Prior Art and R&D

N.A

### Explanation

When the option `--env` or `MEILI_ENV` is set to `production`, the frontend is disabled.

### Impact on documentation

Documentation needs to be updated in:

- The [environment documentation](https://docs.meilisearch.com/guides/advanced_guides/configuration.html#environment)
- The [web interface documentation](https://docs.meilisearch.com/guides/advanced_guides/web_interface.html#web-interface)

## Second Section: Technical Specifications

### Architecture

N.A

### Implementation Details

Pass an argument to `create_app` to conditionally enable the frontend service.

### Corner Cases

TDB

## Third Section: Future possibilities

- Add an opt-in flag to re-enable the frontend in production environment, as suggested per [comment](https://github.com/meilisearch/specifications/pull/8#issuecomment-729676988)
