# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Seq Scan vs Index Scan Comparison

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const SeqScanVsIndexScanDemo = () => {
  const frame = useCurrentFrame();

  const seqScanProgress = interpolate(frame, [0, 120], [0, 100], { extrapolateRight: 'clamp' });
  const indexScanProgress = interpolate(frame, [0, 5], [0, 100], { extrapolateRight: 'clamp' });

  const seqRows = Math.floor(seqScanProgress * 1000000);
  const indexPages = Math.min(Math.floor(indexScanProgress * 0.08), 8);

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Sequential Scan vs Index Scan (100M rows)</h2>

      <div style={{ marginTop: 30 }}>
        <h3 style={{ color: '#e74c3c' }}>Seq Scan: Reading every row</h3>
        <div style={{ background: '#333', height: 30, borderRadius: 4, overflow: 'hidden' }}>
          <div style={{ width: `${seqScanProgress}%`, height: '100%', background: '#e74c3c', transition: 'width 0.1s' }} />
        </div>
        <span>{seqRows.toLocaleString()} / 100,000,000 rows scanned ({(seqScanProgress * 300).toFixed(0)}ms)</span>
      </div>

      <div style={{ marginTop: 30 }}>
        <h3 style={{ color: '#2ecc71' }}>Index Scan: B-tree traversal</h3>
        <div style={{ background: '#333', height: 30, borderRadius: 4, overflow: 'hidden' }}>
          <div style={{ width: `${indexScanProgress}%`, height: '100%', background: '#2ecc71', transition: 'width 0.1s' }} />
        </div>
        <span>{indexPages} / 8 pages read ({(indexScanProgress * 0.5).toFixed(0)}ms)</span>
      </div>
    </div>
  );
};
```
