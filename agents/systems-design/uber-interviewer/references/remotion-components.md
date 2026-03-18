# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Driver Tracking & Matching Animation

```tsx
// Remotion component: UberMatchingDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const UberMatchingDemo = () => {
  const frame = useCurrentFrame();

  // Animation phases
  // 0-30: Drivers moving
  // 30-60: Rider requests, radius expands
  // 60-90: Match found, driver navigates

  const isRequesting = frame > 30;
  const isMatched = frame > 60;

  // Simple map representation
  return (
    <div style={{ width: 600, height: 600, background: '#111', color: 'white', position: 'relative' }}>
      <h2 style={{ textAlign: 'center', padding: 20 }}>Dispatch System</h2>

      {/* City Grid */}
      <div style={{
        position: 'absolute', top: 100, left: 100, width: 400, height: 400,
        border: '1px solid #333',
        backgroundImage: 'linear-gradient(#333 1px, transparent 1px), linear-gradient(90deg, #333 1px, transparent 1px)',
        backgroundSize: '40px 40px'
      }}>

        {/* Rider */}
        <div style={{
          position: 'absolute', top: 200, left: 200,
          width: 16, height: 16, background: '#0070f3', borderRadius: '50%',
          transform: 'translate(-50%, -50%)',
          boxShadow: isRequesting && !isMatched ? `0 0 0 ${(frame - 30) * 2}px rgba(0, 112, 243, 0.2)` : 'none'
        }} />

        {/* Driver 1 (Matches) */}
        <div style={{
          position: 'absolute',
          top: isMatched ? 200 - (90 - frame) : 100 + (frame % 20),
          left: isMatched ? 200 + (90 - frame) : 280,
          width: 16, height: 16, background: '#00ff00', borderRadius: '50%',
          transform: 'translate(-50%, -50%)'
        }} />

        {/* Driver 2 (Ignores) */}
        <div style={{
          position: 'absolute',
          top: 300,
          left: 100 + (frame * 0.5),
          width: 16, height: 16, background: 'gray', borderRadius: '50%',
          transform: 'translate(-50%, -50%)'
        }} />

      </div>

      {/* Status Text */}
      <div style={{ position: 'absolute', bottom: 30, width: '100%', textAlign: 'center', fontSize: 24 }}>
        {!isRequesting ? 'Drivers sending telemetry...' :
         !isMatched ? 'Querying geospatial index...' :
         'Driver dispatched via WebSockets'}
      </div>
    </div>
  );
};
```
