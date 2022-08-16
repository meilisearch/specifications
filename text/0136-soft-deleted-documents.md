# Soft Deleted Documents

## 1. Summary

This specification describes the internals of the soft-deleted documents algorithm.

## 2. Motivation

Deleting documents is extremely slow and can happen when;
- You delete a single document.
- You delete a batch of documents.
- You update one or multiple documents (i.e., the primary key is the same, but the document's content is not the same).

The purpose of the soft-deleted documents feature is to make the deletion of documents almost instantaneous by **not** deleting the document when asked.

## 3. Functional Specification

Instead of deleting the documents, we mark them internally as deleted and then exclude them from all the other algorithms of the engine.
That's fast but takes space; thus, at some point, we need to _really_ delete the soft deleted documents.
This can happen for two reasons;
- When 90% of the total available space is used.
- When 10% of the total space is dedicated to the soft deleted documents.

The idea is good, but there are two little problems;
1. We don't know the size a document really occupies.
  This means we don't know the size used by the soft deleted documents.
  That can be imprecise in the case of a really heterogeneous dataset with large and small documents.
2. We don't know the total available space. The only information available to meilisearch is the `max-index-size` which is by default at 100GB, but meilisearch could be deployed on a smaller disk.

The second point could be a real issue for the case of someone who has very few documents but update them frequently on a small disk without updating the `max-index-size` parameter.
The soft-deleted documents would grow until they use 10GB of disk even though the user only has like 100MB of documents.

## 4. Future possibilities
- Work again on the way to get the size of the disk the `data.ms` is currently running on. This would improve the analytics as well.
- Provide a cli parameter to select how much space can be used to store the soft deleted documents.
  It could be expressed as a real size or in terms of percentage.
- Provide a route to delete the soft deleted documents.
  It could be useful if you **know** you'll have a lot of updates during the day but nothing around midnight, for example.
  It would allow you to clear the soft deleted when meilisearch is not under pressure to ensure all your updates stay fast during the day.
