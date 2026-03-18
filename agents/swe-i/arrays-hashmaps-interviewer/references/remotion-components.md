# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Two Pointers Demo

```tsx
// Remotion component: TwoPointersDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';
import { useState } from 'react';

export const TwoPointersDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const arr = [1, 2, 3, 4, 5, 6];
  const target = 7;

  // Animation: Move pointers based on frame
  const leftPos = Math.min(frame / fps, arr.length - 1);
  const rightPos = arr.length - 1 - Math.min(frame / fps, arr.length - 1);

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 40, padding: 50 }}>
      <div>Target Sum: {target}</div>
      <div style={{ marginTop: 30, display: 'flex', gap: 20 }}>
        {arr.map((num, i) => (
          <div
            key={i}
            style={{
              width: 60,
              height: 60,
              border: '3px solid #333',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              background: i === Math.floor(leftPos) ? '#90EE90' :
                         i === Math.floor(rightPos) ? '#FFB6C1' : 'white'
            }}
          >
            {num}
          </div>
        ))}
      </div>
      <div style={{ marginTop: 20 }}>
        {Math.floor(leftPos) < Math.floor(rightPos) && (
          <div>
            Sum: {arr[Math.floor(leftPos)]} + {arr[Math.floor(rightPos)]} =
            {arr[Math.floor(leftPos)] + arr[Math.floor(rightPos)]}
          </div>
        )}
      </div>
    </div>
  );
};
```
