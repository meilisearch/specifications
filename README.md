# Specifications

This repository provides a template for creating **feature specifications** for MeiliSearch. A feature specification is a written description of a feature that serves as a basis for development, design, and inter-team synchronization.

## Process

When a new feature or product is to be developed, a new pull request is created, and contributors are invited to discuss its content and propose suggestions. The goal of a feature specification is to define the expected behavior on a high level and point out corner cases that need to be addressed.

The person in charge of the PR (the owner) is the person assigned to the PR. This allows for changing ownership. When the PR assignee changes, both new and old assignees should be notified.

#### Start

To start a new specification, it is recommended to write the first draft by filling in only the `Summary` and `Motivation` parts.

The name of the spec must follow the pattern: `PR_number-feature-name.md`. For example, if PR number 12 is about facetting, its spec will be named `0012-facetting.md` or `0012-facetted-search.md`. It will be logical that the names of the PRs will not follow each other. This is not a problem. The fact that the name has the number of PR simplifies the fact that there is no synchronization to choose this number.

#### Draft PR

Once the file is created with the two parts, `Summary` and `Motivation` filled in. It is time to create a draft PR. The PR should remain in draft until the content is, according to the author, ready for comments. However, outsiders can make some recommendations, but these are more in the spirit of helping rather than suggesting changes.

#### Open PR

Once the specification is finished in the owner's eyes, at least the `Summary` and `Motivation` part must be written, he can then switch the PR to open. At that time, it can be commented on, modified, and challenged by these peers. If there is the slightest friction during this process, a discussion will be recommended offline within MeiliSearch. The PR owner will organize this discussion, and at the end of this interview, the owner will clearly describe the report of the meeting in the PR and make any necessary changes.

#### Merged PR

To validate and merge the PR, it must be validated by at least three people.

- One person from the integration team (preferably @curquiza).
- One person from the Core team (preferably @Kerollmops).
- One person from the DevRel team (preferably the person in charge of the documentation).

Once it's done, the differents tracking-issues could be created on the [MeiliSearch](https://github.com/meilisearch/meilisearch) repository.

## Specification description

MeiliSearch's feature specifications are made up of three sections, described below.

### 1. Functional Specification

This first section gives a high level overview of the feature. It should avoid technical language so that it can be understood by a general audience (think user-level). It is broken into five sub-sections, which are as follows:

#### I. Summary

One paragraph explaining the feature from the perspective of a user, e.g. "When I do X I want Y to happen because Z". This paragraph describes the problem and solution simply, without going into detail.

#### II. Motivation

Why develop this feature? What use cases does it support? What is the expected outcome? Add links to related issues and discussions.

#### III. Additional Materials

Discuss and include links to similar features in other tools or products. Share any concept art or visualizations that demonstrate what this proposal might look like in our search API (can be a screenshot of another tool, or a mockup made w/ dev tools). Consider any lessons that can be learned from outside implementations of the feature.

This section is intended to provide readers of your pull request with a fuller picture of the proposed feature by comparing it with similar features in other tools and offering visual aids. If you don't have any examples of similar features being implemented elsewhere, that is fine—your ideas are interesting to us, whether they are brand new or adaptations from other tools.

#### IV. Explanation

Thoroughly explain your feature as if it was already implemented in MeiliSearch and you were teaching another user how to use it. That generally means:

- Introducing (and naming) any new concepts.
- Explaining the feature largely through examples.
- Noting the API for this feature, HTTP, CLI or config.
- Explaining how the user should _think_ about the feature and how it will impact the way they use MeiliSearch (provide concrete examples).
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.

If the changes modify the HTTP API, provide a description of the method, URL, parameters, body, status code, errors, etc...

If it modifies the CLI, provide the env variable name, the argument name, and the description.

This section serves as a user-level guide and should resemble official documentation. Anything that the user may encounter while interacting with the feature should be presented here.

#### V. Impact on Documentation

If the feature requires additions or updates to the documentation, they should be noted here. It's the role of the documentation team to ensure this section of the feature specification is accurate.

### 2. Technical Specifications

This section of the feature specification document has a much narrower audience: the developer(s) that will implement the feature. Its goal is to discuss and clarify how the feature will be developed and implemented. It has three subsections.

#### I. Architecture

This section presents how the new feature will fit into the existing architecture and codebase on an abstract level. It should also define a scope where the feature exists.

#### II. Implementation Details

This section is for precise, practical aspects of implementation, e.g. interfaces, potential code problems, or specific algorithmic choices.

#### III. Corner Cases

Take note of any potential problem areas, undefined behavior, or remaining unanswered questions about behavior here.

### 3. Future Possibilities

This last section includes any related topics or features which are not currently in MeiliSearch and will not be added at this time, but which may affect the proposed feature in the future.
