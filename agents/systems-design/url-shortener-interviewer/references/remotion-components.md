# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Consistent Hashing Animation

```tsx
// Remotion component: ConsistentHashingDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const ConsistentHashingDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Servers on the ring
  const servers = ['Server A', 'Server B', 'Server C'];
  const serverPositions = [0, 120, 240]; // degrees

  // Key being placed
  const keyAngle = (frame * 2) % 360;

  return (
    <div style={{ width: 600, height: 600, margin: '0 auto', position: 'relative' }}>
      {/* Hash ring */}
      <div style={{
        width: 400,
        height: 400,
        borderRadius: '50%',
        border: '4px solid #333',
        position: 'absolute',
        top: 100,
        left: 100
      }}>
        {/* Servers */}
        {servers.map((server, i) => {
          const angle = (serverPositions[i] * Math.PI) / 180;
          const x = 200 + 200 * Math.cos(angle) - 30;
          const y = 200 + 200 * Math.sin(angle) - 15;
          return (
            <div key={server} style={{
              position: 'absolute',
              left: x,
              top: y,
              background: '#4EC9B0',
              padding: '5px 10px',
              borderRadius: 5,
              fontSize: 12,
              fontWeight: 'bold'
            }}>
              {server}
            </div>
          );
        })}

        {/* Key being routed */}
        <div style={{
          position: 'absolute',
          left: 200 + 180 * Math.cos((keyAngle * Math.PI) / 180) - 10,
          top: 200 + 180 * Math.sin((keyAngle * Math.PI) / 180) - 10,
          width: 20,
          height: 20,
          background: '#CE9178',
          borderRadius: '50%',
          transition: 'all 0.1s'
        }} />
      </div>

      <div style={{
        position: 'absolute',
        bottom: 20,
        left: 0,
        right: 0,
        textAlign: 'center',
        fontSize: 18
      }}>
        Key routes to first server clockwise on the ring
      </div>
    </div>
  );
};
```
