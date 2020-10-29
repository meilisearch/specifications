# specifications
specifications of the main features of MeiliSearch

## Process

The goal of the specification is to serve as a basis for development, design
and inter-team synchronization. When a new feature or product is to be
developed, a new pull request is created, and people are invited to discuss
the its content. The goal here is to specify, on a high level the expected
behaviour, but also to point out corner cases that need to be addressed. 

The specifications at MeiliSearch are made up of 3 sections that are described
in the rest of this document.

## First section: Feature Description and Interaction

This first part has a general audience. It should be as little technical as
possible (think user-level). This section contains 4 sub-sections:

### Summary

One paragraph explanation of the feature, formulated as a user story (*"When I
do X I want Y to happen because Z"*). This paragraph explains the problem from
a user perspective.

### Motivation

Why are we doing this? What use cases does it support? What is the expected
outcome? Link to related issues and discussions.

### Prior Art and R&D

Discuss prior art, both the good and the bad, in relation to this proposal. Put
some links about what we can see on others tools, search API, or devtools.

This section is intended to encourage you as an author to think about the
lessons from other tools, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us
whether they are brand new or if it is an adaptation from other tools.

### Explanation

Explain the proposal as if it was already included in MeiliSearch and you were
teaching it to another user. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- The API for this feature, HTTP, CLI or config.
- Explaining how user should *think* about the feature, and how it should impact the way they use MeiliSearch. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.

If the changes modify the HTTP API, provide a description of method, url,
parameters, body, status code, errors, etc... 

If the changes modify the CLI, provide the env variable name, the argument name
and the description.

This serves as a user-level guide. Anything that the user may encounter during
its interaction with the feature should be presented here.

### Impact on documentation

If the feature require that things are added to the documentation, or if
sections of the documentation need to be updated because of this feature, it
should be said here. It's the role of the documentation team to point out the
sections of the documentation that need to be updated.

## Technical Specifications

This section has a much narrower audience: the developer that will implement
the feature. It's goal is to make it as clear as possible what is needed to
actually develop the feature, share knowledge and think about the possibilities.

### Architecture

This section discuss how the new feature should be done on an abstract level,
and how it fits within the rest of the code base. This should define a scope
where the feature exists.

### Implementation Details

Some aspects will need to be made precise, such as interfaces, or specific
algorithmic choices.

### Corner Cases

Some aspects of the development will necessitate special care, they should be
pointed out, and if there are still unanswered question, they belong here too.

## Future possibilities

This last section talks about what has been though of that is related to this
issue, but has been decided not to be done now, and what it means regarding the
feature at hand.
