# Documents Soft Deletion

## 1. Summary

This specification describes the internals of the document soft-deletion algorithm.

## 2. Motivation

Deleting documents is extremely slow and can happen when;
- A user deletes a single document.
- A user deletes a batch of documents.
- A user updates one or multiple documents (i.e., the primary key is the same, but the document's content is not the same).

The purpose of the document soft-deletion feature is to make the deletion of documents almost instantaneous by **not** deleting the document when asked.

## 3. Functional Specification

Instead of deleting the documents, Meilisearch marks them internally as deleted and then excludes them from all the other algorithms of the engine.
That's fast but takes up space; thus, at some point, we need to _really_ delete the soft-deleted documents.

This can happen for two reasons;
1. when there are more soft-deleted documents than regular documents in the database, or
2. when the soft-deleted documents occupy more disk space than a fixed threshold.

Reason (2) presents the drawback that we don't know the precise disk space taken by a document, for technical reasons. Since the information we have is the total size taken by all documents (soft-deleted or not) and the number of documents, we approximate the size of a document to the average size of a document.
This means that if a few outliers are updated/deleted, they can take up much more disk space than the fixed threshold.

## 4. Future Possibilities

- Work again on the way to get the size of the disk the `data.ms` is currently running on. This would improve the analytics as well.
- Provide a CLI parameter to select how much space can be used to store the soft deleted documents.
  - It could be expressed as a real size or in terms of percentage.
- Provide a route to delete the soft-deleted documents.
  - It could be useful if a user **knows** they will have a lot of updates during the day but nothing around midnight, for example.
  - It would allow a user to clear the soft-deleted when Meilisearch is not under pressure to ensure all your updates stay fast during the day.
