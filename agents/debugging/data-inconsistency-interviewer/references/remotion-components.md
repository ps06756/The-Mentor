# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Discrepancy Waterfall Breakdown

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const DiscrepancyWaterfallDemo = () => {
  const frame = useCurrentFrame();

  const factors = [
    { label: 'Dashboard Total', amount: 1200000, color: '#e74c3c', revealFrame: 0 },
    { label: '- Timezone Overlap', amount: -45000, color: '#e67e22', revealFrame: 30 },
    { label: '- Duplicate Events', amount: -80000, color: '#f1c40f', revealFrame: 60 },
    { label: '- Missing Refunds', amount: -25000, color: '#9b59b6', revealFrame: 90 },
    { label: '= Corrected Total', amount: 1050000, color: '#2ecc71', revealFrame: 120 },
  ];

  let runningTotal = 1200000;

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Revenue Discrepancy Breakdown</h2>

      <div style={{ marginTop: 20 }}>
        {factors.map((f, i) => {
          const visible = frame >= f.revealFrame;
          const opacity = interpolate(frame, [f.revealFrame, f.revealFrame + 15], [0, 1], {
            extrapolateLeft: 'clamp',
            extrapolateRight: 'clamp',
          });

          if (i > 0 && i < 4) runningTotal += f.amount;

          return (
            <div key={f.label} style={{ opacity, display: 'flex', justifyContent: 'space-between', padding: '8px 0', borderBottom: '1px solid #333' }}>
              <span style={{ color: f.color }}>{f.label}</span>
              <span style={{ fontFamily: 'monospace' }}>
                ${Math.abs(f.amount).toLocaleString()}
              </span>
            </div>
          );
        })}
      </div>

      {frame >= 130 && (
        <div style={{ marginTop: 20, textAlign: 'center', color: '#2ecc71', fontSize: 18 }}>
          Matches Finance: $1,050,000
        </div>
      )}
    </div>
  );
};
```
