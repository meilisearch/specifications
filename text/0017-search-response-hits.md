- Title: Hits in Search Response
- Start Date:
- Specification PR:
- MeiliSearch Issue: 

# Title

## 1. Feature Description and Interaction

### Summary

As of today, in the `hits` key in the response body upon search the following fields are/may be present: 
- displayed fields
- `_formated` 
- `_matchesInfo`

### Motivation

As this may not be the most intuitive or simple design, lets consider how we could improve it.

The subfields present in the hits should be rediscussed:
- Should they exist
- Should they be renamed
- Should they output differently


### Additional Materials

related issues: meilisearch/transplant#75

// TODO 

### Explanation


### Impact on Documentation

// TODO 

## 2. Technical Specifications

### Architecture
### Implementation Details
### Corner Cases

## 3. Future Possibilities

- Remove `_formatedHits` and apply formating directly on first-level fields.
- Keep like it is
- Rename fields
- Remove `_matchesInfo` or rename it as we don't think anyone use it, or knows about it.