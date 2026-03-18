# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Fan-out on Write vs Read Animation

```tsx
// Remotion component: FanOutDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const FanOutDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Animation phases
  // 0-45: Fan-out on WRITE -- tweet propagates to follower caches
  // 45-90: Fan-out on READ -- follower assembles timeline at read time

  const phase = frame < 45 ? 'write' : 'read';
  const phaseFrame = phase === 'write' ? frame : frame - 45;

  const followers = ['User B', 'User C', 'User D', 'User E'];
  const followees = ['User A', 'User X', 'User Y'];

  return (
    <div style={{ width: 800, height: 600, background: '#0a0a0a', color: 'white', fontFamily: 'monospace', position: 'relative' }}>
      <h2 style={{ textAlign: 'center', padding: 20, color: '#1DA1F2' }}>
        {phase === 'write' ? 'Fan-out on WRITE (Push)' : 'Fan-out on READ (Pull)'}
      </h2>

      {phase === 'write' && (
        <>
          {/* Tweet origin */}
          <div style={{
            position: 'absolute', top: 250, left: 50,
            padding: '10px 20px', background: '#1DA1F2', borderRadius: 8,
            opacity: phaseFrame > 5 ? 1 : phaseFrame / 5
          }}>
            User A tweets
          </div>

          {/* Fan-out arrows and caches */}
          {followers.map((f, i) => {
            const delay = 10 + i * 5;
            const progress = Math.min(1, Math.max(0, (phaseFrame - delay) / 15));
            return (
              <div key={f} style={{
                position: 'absolute',
                top: 120 + i * 80,
                left: 250 + progress * 300,
                padding: '8px 16px',
                background: progress > 0.8 ? '#2ecc71' : '#333',
                borderRadius: 6,
                opacity: progress > 0 ? 1 : 0,
                transition: 'background 0.3s'
              }}>
                {f} cache: [T1, ...]
              </div>
            );
          })}
        </>
      )}

      {phase === 'read' && (
        <>
          {/* Reader */}
          <div style={{
            position: 'absolute', top: 250, left: 600,
            padding: '10px 20px', background: '#e74c3c', borderRadius: 8
          }}>
            User B reads
          </div>

          {/* Followee tweet stores */}
          {followees.map((f, i) => {
            const delay = 5 + i * 5;
            const progress = Math.min(1, Math.max(0, (phaseFrame - delay) / 15));
            return (
              <div key={f} style={{
                position: 'absolute',
                top: 120 + i * 120,
                left: 50 + progress * 200,
                padding: '8px 16px',
                background: progress > 0.8 ? '#f39c12' : '#333',
                borderRadius: 6,
                opacity: progress > 0 ? 1 : 0
              }}>
                {f} tweets
              </div>
            );
          })}
        </>
      )}

      {/* Status */}
      <div style={{ position: 'absolute', bottom: 30, width: '100%', textAlign: 'center', fontSize: 18, color: '#888' }}>
        {phase === 'write'
          ? 'Tweet pushed to each follower cache at write time'
          : 'Timeline assembled from followee stores at read time'}
      </div>
    </div>
  );
};
```
