# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Retry with Jitter Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const RetryJitterDemo = () => {
  const frame = useCurrentFrame();

  // Demonstrating the Thundering Herd problem without jitter vs with jitter
  const withoutJitterY = 100;
  const withJitterY = 250;

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Retries: Exponential Backoff + Jitter</h2>

      {/* Without Jitter (Thundering Herd) */}
      <div style={{ marginTop: 20 }}>
        <h3 style={{ color: '#ce9178' }}>Without Jitter (Fixed Backoff)</h3>
        <div style={{ position: 'relative', height: 60, borderBottom: '1px solid #555' }}>
          <div style={{ position: 'absolute', left: 50, top: 10 }}>[100 Clients Fail]</div>
          <div style={{ position: 'absolute', left: 200, top: 10, color: 'red' }}>[100 Clients Retry at 2s]</div>
          <div style={{ position: 'absolute', left: 400, top: 10, color: 'red' }}>[100 Clients Retry at 4s]</div>
        </div>
      </div>

      {/* With Jitter */}
      <div style={{ marginTop: 50 }}>
        <h3 style={{ color: '#4EC9B0' }}>With Jitter (Randomized Backoff)</h3>
        <div style={{ position: 'relative', height: 100, borderBottom: '1px solid #555' }}>
          <div style={{ position: 'absolute', left: 50, top: 10 }}>[100 Clients Fail]</div>

          {/* Spread out retries */}
          <div style={{ position: 'absolute', left: 150, top: 10, fontSize: 10 }}>Retry</div>
          <div style={{ position: 'absolute', left: 180, top: 30, fontSize: 10 }}>Retry</div>
          <div style={{ position: 'absolute', left: 210, top: 50, fontSize: 10 }}>Retry</div>
          <div style={{ position: 'absolute', left: 250, top: 10, fontSize: 10 }}>Retry</div>
          <div style={{ position: 'absolute', left: 290, top: 30, fontSize: 10 }}>Retry</div>

          <div style={{ position: 'absolute', left: 400, top: 40, color: '#4EC9B0' }}>Load is spread out! Server recovers.</div>
        </div>
      </div>
    </div>
  );
};
```
