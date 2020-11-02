# specifications
Specifications of the main features of MeiliSearch.

## Process

The goal of the specification is to serve as a basis for development, design, and  inter-team synchronization. When a new feature or product is to be developed, a new pull request is created, and people are invited to discuss its content and propose suggestions. The goal here is to specify the expected behavior on a high level and point out corner cases that need to be addressed.

The specifications at MeiliSearch are made up of 3 sections that are described in the rest of this document.

## First section: Feature Description and Interaction

This first part has a general audience. It should be as little technical as possible (think user-level). This section contains 4 sub-sections:

### Summary

One paragraph explains the feature formulated as a user story ("When I do X I want Y to happen because Z"). This paragraph describes the problem from a user perspective.

### Motivation

Why are we doing this? What use cases does it support? What is the expected outcome? Link to related issues and discussions.

### Prior Art and R&D

Discuss prior art, both the good and the bad, concerning this proposal. Put some links about what we can see on other tools, search API, or dev tools.

This section intends to encourage you as an author to think about the lessons from other tools and provide readers of your RFC with a fuller picture. If there is no prior art, that is fine - your ideas are interesting to us, whether they are brand new or adaptation from other tools.

### Explanation

Explain the proposal as if it was already included in MeiliSearch and you were teaching it to another user. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- The API for this feature, HTTP, CLI or config.
- Explaining how the user should _think_ about the feature and how it should impact the way they use MeiliSearch. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.

If the changes modify the HTTP API, provide a description of the method, URL, parameters, body, status code, errors, etc...

If it modifies the CLI, provide the env variable name, the argument name, and the description.

This serves as a user-level guide. Anything that the user may encounter during its interaction with the feature should be presented here.

### Impact on documentation

If the feature requires additions to the documentation or if sections of the documentation need to be updated because of this feature, it should be mentioned here. It's the role of the documentation team to point out the sections of the documentation that need to be updated.

## Second Section: Technical Specifications

This section has a much narrower audience: the developer that will implement the feature. Its goal is to make it as clear as possible to develop the feature, share knowledge, and think about the possibilities.

### Architecture

This section presents how the new feature should be done on an abstract level and how it fits within the codebase. This should define a scope where the feature exists.

### Implementation Details

Some aspects will need to be made precise, such as interfaces or specific algorithmic choices.

### Corner Cases

Some aspects of the development will necessitate special care, they should be pointed out, and if there are still unanswered questions, they belong here too.

## Third Section: Future possibilities

This last section talks about what has been thought of related to this issue, but has been decided not to be done now, and what it means regarding the feature at hand.
