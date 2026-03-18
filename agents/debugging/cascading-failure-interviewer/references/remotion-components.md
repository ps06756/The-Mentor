# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Cascading Failure Propagation

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const CascadingFailureDemo = () => {
  const frame = useCurrentFrame();

  const services = [
    { name: 'Bank API', failFrame: 0, color: '#e74c3c', y: 300 },
    { name: 'Payment', failFrame: 15, color: '#e67e22', y: 240 },
    { name: 'Order', failFrame: 45, color: '#f1c40f', y: 180 },
    { name: 'Cart', failFrame: 70, color: '#9b59b6', y: 120 },
    { name: 'Frontend', failFrame: 90, color: '#3498db', y: 60 },
  ];

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Cascading Failure Propagation</h2>

      {services.map((svc) => {
        const isFailed = frame >= svc.failFrame;
        const opacity = interpolate(frame, [svc.failFrame, svc.failFrame + 10], [0.3, 1], {
          extrapolateLeft: 'clamp',
          extrapolateRight: 'clamp',
        });

        return (
          <div key={svc.name} style={{ position: 'absolute', left: 50, top: svc.y, display: 'flex', alignItems: 'center', gap: 10 }}>
            <div style={{
              width: 120, height: 36, borderRadius: 6,
              background: isFailed ? svc.color : '#2d2d2d',
              border: `2px solid ${isFailed ? svc.color : '#555'}`,
              display: 'flex', alignItems: 'center', justifyContent: 'center',
              opacity: isFailed ? opacity : 0.5,
              fontSize: 13,
            }}>
              {svc.name}
            </div>
            <span style={{ fontSize: 12, color: isFailed ? '#e74c3c' : '#4CAF50' }}>
              {isFailed ? 'DEGRADED' : 'Healthy'}
            </span>
          </div>
        );
      })}
    </div>
  );
};
```
