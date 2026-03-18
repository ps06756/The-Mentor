# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Vector Clocks Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const VectorClockDemo = () => {
  const frame = useCurrentFrame();

  // A: [1,0,0], B: [0,0,0], C: [0,0,0]
  // Event 1: A updates -> [1,0,0]
  // Event 2: A sends to B -> B updates -> [1,1,0]
  // Event 3: B sends to C -> C updates -> [1,1,1]
  // Event 4: Concurrent! A updates [2,0,0], C updates [1,1,2]

  const step = Math.floor(frame / 30);

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Vector Clocks: Causality Tracking</h2>

      <div style={{ display: 'flex', justifyContent: 'space-around', marginTop: 50 }}>

        <div style={{ textAlign: 'center' }}>
          <h3>Node A</h3>
          <div style={{ padding: 10, border: '1px solid #569CD6', fontFamily: 'monospace' }}>
            {step < 1 ? "[0,0,0]" : step < 4 ? "[1,0,0]" : "[2,0,0]"}
          </div>
        </div>

        <div style={{ textAlign: 'center' }}>
          <h3>Node B</h3>
          <div style={{ padding: 10, border: '1px solid #CE9178', fontFamily: 'monospace' }}>
            {step < 2 ? "[0,0,0]" : "[1,1,0]"}
          </div>
        </div>

        <div style={{ textAlign: 'center' }}>
          <h3>Node C</h3>
          <div style={{ padding: 10, border: '1px solid #DCDCAA', fontFamily: 'monospace' }}>
            {step < 3 ? "[0,0,0]" : step < 4 ? "[1,1,1]" : "[1,1,2]"}
          </div>
        </div>

      </div>

      <div style={{ marginTop: 50, textAlign: 'center', padding: 10 }}>
        {step === 4 && (
          <p style={{ color: 'orange' }}>
            Conflict! A [2,0,0] and C [1,1,2] are concurrent.<br/>
            Neither vector is strictly greater than the other. System must resolve via client logic or LWW.
          </p>
        )}
      </div>
    </div>
  );
};
```
