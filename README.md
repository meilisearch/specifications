# Specifications

This repository provides a template for creating **feature specifications** for MeiliSearch. A feature specification is a written description of a feature that serves as a basis for development, design, and inter-team synchronization. 

A **Merged specification** represents a MeiliSearch feature that is implemented or ready to be.

---

## Specification Workflow

When a new feature or product is to be developed, a new pull request is created, and contributors are invited to discuss its content and propose suggestions. The goal of a feature specification is to define the expected behavior on a high level and point out corner cases that need to be addressed.

The person in charge of the PR (the owner) is the person assigned to the PR. This allows for changing ownership. When the PR assignee changes, both new and old assignees should be notified.

MeiliSearch's feature specifications flow is made up of 5 states, described below.

### Introduction State

To start a new specification, it is recommended to write the first draft by filling in only the `Summary` and `Motivation` parts. 

The name of the spec file must follow the pattern: `PR_number-feature-name.md`. After the pull request creation, the owner should update the specification filename. For example, if PR number 12 is about facetting, its spec will be named `0012-facetting.md`.

> Note that a pull request not strictly dealing about a specification conception will be tagged as `Not A Spec`.

### Draft State

Once the file is created with the two parts, `Summary` and `Motivation` filled in. It is time to create a draft PR. The PR should remain in draft state until the content is, according to the author, ready for comments. However, outsiders can make some recommendations, but these are more in the spirit of helping rather than suggesting changes.

### Open State

Once the specification is ready for comments in the owner's eyes, the owner can then switch the PR to open. At that time, it can be commented on, modified, challenged by these peers. If there is the slightest friction during this process, a discussion will be recommended offline within MeiliSearch. The PR owner will organize this discussion, and at the end of this interview, the owner will make any necessary changes.

At this step, the PR should be primarily tagged as `In Progress`.

Note that if the pull request does not satisfy Open State conditions it will go back to the previous step above (***Draft State***).

### Review State

The PR should now be tagged as `RFR (Ready For Review)` to enter this stage.

To validate and merge the PR, it must be validated by at least three people.

- One person from the Integration team (preferably @curquiza).
- One person from the Core team (preferably @Kerollmops).
- One person from the DevRel team (preferably the person in charge of the documentation).

Note that if the specification does not satisfy the merging conditions, it will go back to the previous step above (***Open State***) since it's lacking some pieces of information in the reviewer's eyes.

### Merge State

The differents tracking-issues could be created on concerned repositories.

Created issues should:  
- `Spec Related` label has to be added to the issues.
- Issues have to mention the spec in their description.
- Issues must absolutely link to the PRs that resolve the spec, so that we can easily track them.

In order to keep track of technical changes concerning the specification, delivery team should update the `MeiliSearch Tracking-issues` part of the specification. 

Once it's done, the specification is accepted and merged to the main branch.

## Specification Revision

Any MeiliSearch's specification can change over time due to diverse factors. Sometimes related to change made on another specification concerning core engine code that could impact it, or simply product enhancement.

In this precise case, a new PR is created and should follow the specification workflow described above.

---

## Specification Description and Interaction

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

#### VI. Impact on SDKs

If the feature requires additions or updates to the SDks, they should be noted here. It's the role of the integration team to ensure this section of the feature specification is accurate.

### 2. Technical Aspects

This section of the feature specification document has a much narrower audience: the developer(s) that will implement the feature. Its goal is to discuss and clarify how the feature will be developed and implemented when its not a trivial concern.

> Internal technical teams may use this part to keep track of technical brainstorms, related proto code or basecode examples to clear the technical details of the specification.

At this time, the specification process is not enforcing a way to describe technical aspects. It can change in the future if we think that it is needed to ease the workflow of a specification creation.

When writing technical details, we recommend describing practical aspects of implementation, e.g. interfaces, potential code problems, or specific algorithmic choices.

### 3. Future Possibilities

This last section includes any related topics or features which are not currently in MeiliSearch and will not be added at this time, but which may affect the proposed feature in the future.
