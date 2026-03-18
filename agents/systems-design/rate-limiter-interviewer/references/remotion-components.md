# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Distributed Race Condition Demo

```tsx
// Remotion component: RateLimitRaceDemo.tsx
import { useCurrentFrame } from 'remotion';

export const RateLimitRaceDemo = () => {
  const frame = useCurrentFrame();

  const thread1Y = Math.min(100 + frame * 2, 250);
  const thread2Y = Math.min(100 + (frame > 20 ? (frame - 20) * 2 : 0), 250);

  const isConflict = frame > 80;

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Race Condition in Rate Limiting</h2>

      <div style={{ display: 'flex', justifyContent: 'space-around', marginTop: 50 }}>
        {/* Gateway 1 */}
        <div style={{ width: 150, textAlign: 'center' }}>
          <h3>API Gateway 1</h3>
          <div style={{ marginTop: 20, color: '#4EC9B0' }}>
            {frame > 30 ? "GET count (5)" : ""}
            <br/>
            {frame > 60 ? "count + 1 = 6" : ""}
            <br/>
            {frame > 80 ? "SET count = 6" : ""}
          </div>
        </div>

        {/* Central Redis */}
        <div style={{ width: 150, textAlign: 'center', border: '2px solid #ce9178', padding: 10 }}>
          <h3>Redis</h3>
          <h2>Count: {isConflict ? 6 : 5}</h2>
          <p style={{ color: 'red' }}>{isConflict ? "Both threads set to 6! Missed a request." : ""}</p>
        </div>

        {/* Gateway 2 */}
        <div style={{ width: 150, textAlign: 'center' }}>
          <h3>API Gateway 2</h3>
          <div style={{ marginTop: 20, color: '#569CD6' }}>
            {frame > 40 ? "GET count (5)" : ""}
            <br/>
            {frame > 70 ? "count + 1 = 6" : ""}
            <br/>
            {frame > 90 ? "SET count = 6" : ""}
          </div>
        </div>
      </div>

      <div style={{ textAlign: 'center', marginTop: 40, color: '#DCDCAA' }}>
        Solution: Use Redis INCR or Lua scripts for atomicity.
      </div>
    </div>
  );
};
```
