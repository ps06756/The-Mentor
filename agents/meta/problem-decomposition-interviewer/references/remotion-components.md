# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of the Problem-Solving Funnel. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Problem-Solving Funnel Demo

```tsx
// Remotion component: ProblemSolvingFunnelDemo.tsx
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const ProblemSolvingFunnelDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const layers = [
    { label: '1. UNDERSTAND', color: '#4A90D9', width: 100 },
    { label: '2. PLAN', color: '#50B86C', width: 80 },
    { label: '3. CODE', color: '#F5A623', width: 60 },
    { label: '4. TEST', color: '#D0021B', width: 40 },
  ];

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 28, padding: 60, background: '#1a1a2e', color: '#eee' }}>
      <div style={{ fontSize: 36, marginBottom: 40, textAlign: 'center' }}>
        The Problem-Solving Funnel
      </div>
      {layers.map((layer, i) => {
        const revealAt = i * fps;
        const opacity = interpolate(frame, [revealAt, revealAt + fps / 2], [0, 1], {
          extrapolateRight: 'clamp',
        });
        const translateY = interpolate(frame, [revealAt, revealAt + fps / 2], [20, 0], {
          extrapolateRight: 'clamp',
        });
        return (
          <div
            key={i}
            style={{
              opacity,
              transform: `translateY(${translateY}px)`,
              display: 'flex',
              justifyContent: 'center',
              marginBottom: 12,
            }}
          >
            <div
              style={{
                width: `${layer.width}%`,
                padding: '16px 24px',
                background: layer.color,
                borderRadius: 8,
                textAlign: 'center',
                fontWeight: 'bold',
              }}
            >
              {layer.label}
            </div>
          </div>
        );
      })}
    </div>
  );
};
```
