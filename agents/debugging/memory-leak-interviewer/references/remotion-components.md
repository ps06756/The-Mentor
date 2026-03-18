# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Memory Growth Over Time

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const MemoryLeakGrowthDemo = () => {
  const frame = useCurrentFrame();

  const memoryGB = interpolate(frame, [0, 240], [2, 10], { extrapolateRight: 'clamp' });
  const gcReclaimed = interpolate(frame, [0, 240], [1.5, 0.2], { extrapolateRight: 'clamp' });
  const barHeight = interpolate(memoryGB, [0, 10], [0, 300], { extrapolateRight: 'clamp' });
  const isOOM = memoryGB >= 9.8;

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Service Memory: Linear Growth (Leak)</h2>

      <div style={{ display: 'flex', alignItems: 'flex-end', justifyContent: 'center', height: 320 }}>
        <div style={{ display: 'flex', flexDirection: 'column', alignItems: 'center' }}>
          <span style={{ color: isOOM ? '#e74c3c' : '#3498db' }}>
            {isOOM ? 'OOM KILLED!' : `${memoryGB.toFixed(1)} GB`}
          </span>
          <div style={{
            width: 80,
            height: barHeight,
            background: isOOM ? '#e74c3c' : `rgb(${Math.floor(memoryGB * 25)}, ${Math.floor(180 - memoryGB * 15)}, 100)`,
            borderRadius: 4,
            transition: 'height 0.1s',
          }} />
          <span style={{ marginTop: 4, fontSize: 12 }}>Heap Used</span>
        </div>
        <div style={{ marginLeft: 40, fontSize: 14, color: '#888' }}>
          <div>GC reclaims: {gcReclaimed.toFixed(1)} GB/cycle</div>
          <div style={{ color: '#f39c12', marginTop: 8 }}>
            {gcReclaimed < 0.5 && 'Warning: GC effectiveness declining'}
          </div>
        </div>
      </div>
    </div>
  );
};
```
