# specifications
specifications of the main features of meilisearch

## Process

The goal of the specification is to serve as a basis for developpement, design
and inter-team synchronization. When a new feature or product is to be
developped, a new pull request is created, and people are invited to discuss
the its content. The goal here is to specify, on a high level the expected
behaviour, but also to point out corner cases that need to be addressed. A spec
contains multiple sections A spec contains multiple sections. Most of them are
mandatory, and a spec shouldn't be merged if all of its mandatory sections are
not filled. These sections are the following:

- Overview: Explain in plain words what the feature is about, what it tries to
  achieve/address, link to eventual issues and other considerations.
- Research Notes: This section discusses the eventual researches that have been
  done on the topis, link to usefull resources (with explainations), and to
  competition study if relevant.
- Corner Cases: All considerations concerning the corner cases of the feature
  use, technical limitations, etc...
- API design: Be the API internal or external, it's design is discussed in this sections.
- Documentation(optional): if the feature has an impact on the documentation,
  the sections that need to be changed should be present here.
- Implentation guidelines (Optional): If specific implementation decisions have
  been made, thwey are discusses here.
