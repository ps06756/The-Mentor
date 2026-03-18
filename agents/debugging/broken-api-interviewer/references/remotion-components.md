# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Error Rate Spike Timeline

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const ErrorRateSpikeDemo = () => {
  const frame = useCurrentFrame();

  const errorRate = interpolate(frame, [0, 30, 60, 90], [0.8, 2.1, 22.3, 30.1], {
    extrapolateRight: 'clamp',
  });

  const barWidth = interpolate(errorRate, [0, 35], [0, 500], { extrapolateRight: 'clamp' });

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Checkout API: 5xx Error Rate After Deploy</h2>

      <div style={{ marginTop: 40 }}>
        <div style={{ display: 'flex', alignItems: 'center', gap: 10 }}>
          <span style={{ width: 80 }}>14:00</span>
          <div style={{ width: barWidth, height: 30, background: errorRate > 10 ? '#e74c3c' : '#f39c12', borderRadius: 4 }} />
          <span>{errorRate.toFixed(1)}%</span>
        </div>
        <div style={{ marginTop: 10, color: '#888' }}>
          {frame > 30 && '^ Deploy v2.4.0 happened here'}
        </div>
        <div style={{ marginTop: 20, color: frame > 60 ? '#e74c3c' : '#888' }}>
          {frame > 60 && 'PAGERDUTY: Checkout API error rate exceeds threshold (1%)'}
        </div>
      </div>
    </div>
  );
};
```
