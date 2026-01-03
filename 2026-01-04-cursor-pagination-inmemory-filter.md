# Cursor-Based Pagination with In-Memory Filtering

*Learned: 2026-01-04*

## Context

Building mobile notifications list with title filter that's applied in-memory after Firestore query. Fixed edge case where empty filtered results broke pagination continuity.

## Insight

When implementing pagination with filters that can't be pushed to the database (like case-insensitive substring search in Firestore), the cursor must track **database position**, not filtered results position.

### The Problem

```typescript
// ❌ Wrong: cursor based on last filtered result
notifications.push(notification);
lastFetchedDoc = { receivedAt: data.receivedAt, id: docSnap.id };
// ...
result.nextCursor = encodeCursor(lastFetchedDoc.receivedAt, lastFetchedDoc.id);
```

If no records pass the filter, `lastFetchedDoc` is undefined → no cursor → user can't continue loading even though more data exists.

### The Fix

```typescript
// ✅ Correct: cursor based on last DB position
for (const docSnap of docs) {
  // filter and collect...
}
// Track DB position separately from filtered results
if (docs.length > 0) {
  const lastDoc = docs[docs.length - 1];
  currentCursor = encodeCursor(lastDoc.receivedAt, lastDoc.id);
}
// Use DB position for next page
if (hasMoreInDb) {
  result.nextCursor = currentCursor;
}
```

### Key Principle

The pagination cursor represents "where to continue reading from the source" — this is independent of how many results passed your filter. This pattern applies to any system where filtering happens after fetching (Elasticsearch post-filters, API aggregations, etc.).

## Sources

- Learned from implementation experience
