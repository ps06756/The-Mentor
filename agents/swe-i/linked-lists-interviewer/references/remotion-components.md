# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Linked List Reversal Demo

```tsx
// Remotion component: LinkedListReversalDemo.tsx
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const LinkedListReversalDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const nodes = [1, 2, 3, 4, 5];
  const stepDuration = fps * 1.5;
  const currentStep = Math.floor(frame / stepDuration);
  const stepProgress = (frame % stepDuration) / stepDuration;

  // Nodes that have been reversed so far
  const reversedCount = Math.min(currentStep, nodes.length);

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 32, padding: 50 }}>
      <div style={{ marginBottom: 20 }}>Reverse Linked List - Step {Math.min(currentStep + 1, nodes.length)}</div>
      <div style={{ display: 'flex', alignItems: 'center', gap: 10 }}>
        {nodes.map((node, i) => {
          const isReversed = i < reversedCount;
          const isActive = i === reversedCount && currentStep < nodes.length;
          const arrowOpacity = isReversed
            ? interpolate(stepProgress, [0, 0.5], [0, 1], { extrapolateRight: 'clamp' })
            : 1;

          return (
            <div key={i} style={{ display: 'flex', alignItems: 'center' }}>
              <div
                style={{
                  width: 50,
                  height: 50,
                  border: `3px solid ${isActive ? '#FF6B6B' : isReversed ? '#4ECDC4' : '#333'}`,
                  borderRadius: 8,
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center',
                  background: isActive ? '#FFE66D' : isReversed ? '#E0F7FA' : 'white',
                }}
              >
                {node}
              </div>
              {i < nodes.length - 1 && (
                <div style={{ margin: '0 5px', opacity: arrowOpacity }}>
                  {isReversed ? '\u2190' : '\u2192'}
                </div>
              )}
            </div>
          );
        })}
      </div>
      <div style={{ marginTop: 30, fontSize: 24 }}>
        {currentStep >= nodes.length
          ? 'Reversal complete!'
          : `Moving pointer: node ${nodes[reversedCount]}`}
      </div>
    </div>
  );
};
```
