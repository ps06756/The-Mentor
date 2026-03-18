# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## HeapInsertionDemo

```tsx
// Remotion component: HeapInsertionDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const HeapInsertionDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const insertions = [10, 5, 15, 3, 8, 12, 1];
  const step = Math.min(Math.floor(frame / (fps * 2)), insertions.length - 1);

  // Simulate min-heap state after each insertion
  const getHeapState = (upToStep: number) => {
    const heap: number[] = [];
    for (let s = 0; s <= upToStep; s++) {
      heap.push(insertions[s]);
      let i = heap.length - 1;
      while (i > 0) {
        const parent = Math.floor((i - 1) / 2);
        if (heap[parent] > heap[i]) {
          [heap[parent], heap[i]] = [heap[i], heap[parent]];
          i = parent;
        } else {
          break;
        }
      }
    }
    return heap;
  };

  const heap = getHeapState(step);

  // Render heap as a tree using level-order positions
  const getLevel = (index: number) => Math.floor(Math.log2(index + 1));
  const maxLevel = heap.length > 0 ? getLevel(heap.length - 1) : 0;

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 28, padding: 50 }}>
      <div style={{ marginBottom: 20 }}>
        Min-Heap Insertion: inserting {insertions[step]}
      </div>
      <div style={{ position: 'relative', width: 600, height: 300 }}>
        {heap.map((val, i) => {
          const level = getLevel(i);
          const posInLevel = i - (Math.pow(2, level) - 1);
          const nodesInLevel = Math.pow(2, level);
          const x = ((posInLevel + 0.5) / nodesInLevel) * 600;
          const y = level * 80 + 30;
          return (
            <div
              key={i}
              style={{
                position: 'absolute',
                left: x - 25,
                top: y - 25,
                width: 50,
                height: 50,
                borderRadius: '50%',
                border: '3px solid #333',
                display: 'flex',
                alignItems: 'center',
                justifyContent: 'center',
                background: i === heap.length - 1 ? '#FFD700' : '#90EE90',
              }}
            >
              {val}
            </div>
          );
        })}
      </div>
      <div style={{ marginTop: 20 }}>
        Array: [{heap.join(', ')}]
      </div>
    </div>
  );
};
```
