# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Deployment Error Rate Correlation

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const DeployErrorCorrelationDemo = () => {
  const frame = useCurrentFrame();

  const timeMinutes = interpolate(frame, [0, 150], [0, 60], { extrapolateRight: 'clamp' });
  const deployFrame = 37; // 14:00 in the timeline

  const errorRate = frame < deployFrame
    ? 0.1
    : interpolate(frame, [deployFrame, deployFrame + 20, deployFrame + 40], [0.1, 8, 15], {
        extrapolateRight: 'clamp',
      });

  const barWidth = interpolate(errorRate, [0, 16], [0, 400], { extrapolateRight: 'clamp' });
  const isDeployed = frame >= deployFrame;

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Error Rate vs Deploy Time</h2>

      <div style={{ marginTop: 40, position: 'relative' }}>
        <div style={{ display: 'flex', alignItems: 'center', gap: 10 }}>
          <span style={{ width: 60, fontSize: 12 }}>
            {`${Math.floor(13.5 + timeMinutes / 60)}:${String(Math.floor((timeMinutes % 60) * 1)).padStart(2, '0')}`}
          </span>
          <div style={{
            width: barWidth,
            height: 24,
            background: errorRate > 5 ? '#e74c3c' : errorRate > 1 ? '#f39c12' : '#2ecc71',
            borderRadius: 4,
            transition: 'width 0.05s',
          }} />
          <span style={{ fontSize: 14 }}>{errorRate.toFixed(1)}%</span>
        </div>

        {isDeployed && (
          <div style={{ marginTop: 15, color: '#e74c3c', fontSize: 13 }}>
            ^ v2.4.0 deployed at 14:00 — error spike begins at 14:15
          </div>
        )}

        <div style={{ marginTop: 30, padding: 10, background: '#2d2d2d', borderRadius: 4, fontSize: 13 }}>
          {!isDeployed && 'Waiting for deploy...'}
          {isDeployed && errorRate < 5 && 'Deploy in progress... monitoring error rates'}
          {errorRate >= 5 && errorRate < 10 && 'WARNING: Error rate increasing abnormally'}
          {errorRate >= 10 && 'ALERT: Error rate exceeds threshold. Rollback decision needed.'}
        </div>
      </div>
    </div>
  );
};
```
