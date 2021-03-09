- Title: Get Document
- Specification PR: #17

# Title

## 1. Feature Description and Interaction

### Summary

Get one document route: `GET /indexes/:indexUid/documents/:documentsId` has no parameters as of now. Which one should be added?

### Motivation

The parameter `attributesToRetrieve` impacting the hit body used in `GET /indexes/:indexUid/documents/`(get documents) is not present in `GET /indexes/:indexUid/documents/:documentsId` (get one document)

### Additional Materials

meilisearch/transplant/#52

### Explanation

// TODO

### Impact on Documentation

// TODO

## 2. Technical Specifications

### Architecture
### Implementation Details
// TODO
### Corner Cases

## 3. Future Possibilities

- Add `attributesToRetrieve` in  `GET /indexes/:indexUid/documents/:documentsId`