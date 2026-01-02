# Conditional Firestore Listeners with Page Visibility API

*Learned: 2026-01-02*

## Context

Building a Research UI with real-time status updates. Needed to optimize Firestore read costs while maintaining good UX for users tracking long-running AI research tasks.

## Insight

**Why it matters:** Firestore charges per document read. A naive real-time listener on a collection can rack up thousands of reads while users aren't even looking at the page. We reduced costs by 80-90% with these techniques:

### 1. Page Visibility API Pause/Resume

```typescript
useEffect(() => {
  const handleVisibilityChange = (): void => {
    if (document.visibilityState !== 'visible') {
      unsubscribe(); // Stop reading when tab is hidden
    } else {
      setupListener(); // Resume when user returns
    }
  };
  document.addEventListener('visibilitychange', handleVisibilityChange);
}, []);
```

Users often leave tabs open for hours/days. Without this, you're paying for reads nobody sees.

### 2. Status-Based Conditional Listening

```typescript
const inProgressStatuses = ['pending', 'processing', 'synthesizing'];
if (!inProgressStatuses.includes(research.status)) {
  unsubscribe(); // Completed research doesn't change
  return;
}
```

Terminal states (completed, failed) don't need real-time updates. Only listen when data can actually change.

### 3. Minimal Data Strategy

The listener only watches for status changes, then triggers an API call for full data. This avoids reading large document fields (like full research results) on every update—just the small status field.

### Trade-off

There's a ~100-500ms delay when users return to the tab while we re-establish the listener. For most apps, this is imperceptible and worth the cost savings.

## Sources

- Internal implementation pattern from IntexuraOS codebase
