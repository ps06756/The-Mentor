# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Recursion Call Stack Demo

```tsx
// Remotion component: RecursionCallStackDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const RecursionCallStackDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const calls = [
    { label: 'factorial(4)', result: '24', depth: 0 },
    { label: 'factorial(3)', result: '6', depth: 1 },
    { label: 'factorial(2)', result: '2', depth: 2 },
    { label: 'factorial(1)', result: '1', depth: 3 },
  ];

  const activeIndex = Math.min(Math.floor(frame / fps), calls.length - 1);
  const isUnwinding = frame / fps > calls.length;
  const unwindIndex = isUnwinding
    ? calls.length - 1 - Math.min(Math.floor(frame / fps - calls.length), calls.length - 1)
    : calls.length;

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 28, padding: 50 }}>
      <div style={{ fontSize: 36, marginBottom: 30 }}>Call Stack</div>
      {calls.map((call, i) => {
        const visible = i <= activeIndex;
        const resolved = isUnwinding && i >= unwindIndex;
        return visible ? (
          <div key={i} style={{ marginLeft: call.depth * 40, padding: 10, border: '2px solid #333', marginBottom: 4 }}>
            {call.label} {resolved && `-> return ${call.result}`}
          </div>
        ) : null;
      })}
    </div>
  );
};
```
