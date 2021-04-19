# MeiliSearch Development Workflow

MeiliSearch is an open soure project. 

This document provides a general overview of the workflow for its development. 

This document covers only the workflow for the [main project](meilisearch/MeiliSearch).

[TOC]

## Governance

### MeiliSearch and Meili (the company)
MeiliSearch is supported and developed mostly by Meili, the company.

The project is not community-led. Meili is trying to gather and build a community around the open-source project in order to support it development and sustainability. 

## 🔮 Roadmap
The MeiliSearch roadmap is public on [roadmap.meilisearch.com](https://roadmap.meilisearch.com).
Anyone can vote for future features or submit an new idea. The roadmap is moderated and prioritized by MeiliSearch CEO.
When an idea on the roadmap is prioritized it will enter in the development workflow.

> __Note__
> Not all specifications must be born out of a card on the roadmap and not all roadmap cards will be transformed into specifications.


## 🎼 Development Workflow
Many changes including bug fixes or documentation improvements can be implemented and reviewed via the normal GitHub pull request workflow which won't be covered in this document.

Some changes are more substantial and we ask these to be put through a design process and produce consensus among the MeiliSearch community  and teams. 
As a rule of thumb, any changes that impact in any way the user of MeiliSearch (without it being a bugfix) should follow this workflow.

### 📝 Specifications (or RFCs) process

We use specification instead of RFCs in our context because RFCs are more open to discussion (comments) than specifications. As MeiliSearch is still a young project and the founding team have a pretty clear idea about where MeiliSearch is heading we hope this process will help us to be better at specifying MeiliSearch rather than having long discussions about such design or such feature.

That being said, this repository is public because anyone is more than welcome to participate!

---
A specification is a document that lives in the [meilisearch/specifications](meilisearch/specifications) repository:
- A specification open to discussion is materialized as an open pull request.

- An "active" specification is materialized as a merged specification the main branch.
    - _Once a specification is "active", its authors or other contributors may implement and submit the feature as a pull request to the MeiliSearch repository. Being "active" does not mean that the feature will be utimately merged; it does mean that in principle all major stakeholders have agreed to the feature and are amenable to merging it._

- An "implemented" specification define the current behavior of the search engine and must be marked as "implemented".

### 🏗 Implementation process
When a specification is "active", it can be implemented by any contributor to the repository.
If a specification cannot be implemented in one pull request, a tracking issue linked to the specification must be written in order to trackm down the different pull request needed for that specification.

Every pull request on the repository must be linked to a tracking issue or to a specification when possible.
Regularly, pull request and issues on the main repository should be submitted for sorting in order to keep these manageable.

### 🎇 Documentation process
Documentation can be written while the implementation is being developed.
Anyone that submits a pull request to the repository has to ensure that a matching pull request is being submitted to the documentation repository.
The documentation can be written either by the documentation team or the pull request author and has to be approved by both before being merged.

Documentation must be writable from the content of the pull request, the tracking issue, or the specification.
