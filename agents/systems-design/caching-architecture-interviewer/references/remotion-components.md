# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Cache Consistency Race Condition Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const CacheRaceDemo = () => {
  const frame = useCurrentFrame();

  // Thread A (Writer) updates DB to V2, tries to delete cache
  // Thread B (Reader) reads from DB (V2), writes to cache

  const writerY = frame < 40 ? 100 : 300;
  const readerY = frame > 20 && frame < 60 ? 100 : 300;

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Race Condition: Why we DELETE, not SET</h2>

      <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: 30 }}>
        {/* Thread A (Writer) */}
        <div style={{ width: 180, border: '1px solid #ce9178', padding: 10 }}>
          <h3 style={{ color: '#ce9178' }}>Thread A (Write)</h3>
          <ul style={{ fontSize: 14 }}>
            <li style={{ color: frame > 10 ? '#fff' : '#555' }}>1. UPDATE DB (Val=2)</li>
            <li style={{ color: frame > 60 ? '#fff' : '#555' }}>2. SET Cache (Val=2)</li>
          </ul>
        </div>

        {/* Thread B (Reader) */}
        <div style={{ width: 180, border: '1px solid #569cd6', padding: 10 }}>
          <h3 style={{ color: '#569cd6' }}>Thread B (Read)</h3>
          <ul style={{ fontSize: 14 }}>
            <li style={{ color: frame > 20 ? '#fff' : '#555' }}>1. Cache Miss!</li>
            <li style={{ color: frame > 30 ? '#fff' : '#555' }}>2. READ DB (Val=2)</li>
            <li style={{ color: frame > 80 ? '#fff' : '#555' }}>3. SET Cache (Val=2)</li>
          </ul>
        </div>
      </div>

      <div style={{ marginTop: 20, textAlign: 'center', padding: 10, background: '#333' }}>
        <h3>State</h3>
        <p>Database: {frame > 10 ? 'Val=2' : 'Val=1'}</p>
        <p>Cache: {frame < 60 ? '(Empty)' : frame > 60 && frame < 80 ? 'Val=2 (Thread A)' : 'Val=2 (Thread B)'}</p>

        {frame > 80 && (
          <p style={{ color: 'orange', fontSize: 14 }}>
            If Thread A sets V2, and Thread B reads V1 (due to lag) and sets V1 *after* Thread A... the cache holds stale V1 forever!
            Always DELETE the cache key on writes. Let the next reader populate it.
          </p>
        )}
      </div>
    </div>
  );
};
```
