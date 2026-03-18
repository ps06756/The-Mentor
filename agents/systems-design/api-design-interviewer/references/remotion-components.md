# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Offset vs Cursor Pagination Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const PaginationDemo = () => {
  const frame = useCurrentFrame();

  // Data: [A, B, C, D, E]
  // Page 1: [A, B]
  // *New item Z inserted at top* -> [Z, A, B, C, D, E]
  // Page 2 (Offset=2): [B, C] -> BUG! Saw B twice.
  // Page 2 (Cursor=B): [C, D] -> Correct.

  const step = Math.floor(frame / 40);

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Offset vs Cursor Pagination</h2>

      <div style={{ display: 'flex', justifyContent: 'space-around', marginTop: 20 }}>

        {/* DB State */}
        <div style={{ width: 120 }}>
          <h4 style={{ borderBottom: '1px solid #555' }}>DB Items</h4>
          <ul style={{ listStyle: 'none', padding: 0, fontFamily: 'monospace' }}>
            {step >= 1 && <li style={{ color: '#4EC9B0' }}>Z (New!)</li>}
            <li>A</li>
            <li style={{ background: step === 0 ? '#333' : 'transparent' }}>B</li>
            <li style={{ background: step === 2 ? '#555' : 'transparent' }}>C</li>
            <li style={{ background: step === 2 ? '#555' : 'transparent' }}>D</li>
            <li>E</li>
          </ul>
        </div>

        {/* Offset */}
        <div style={{ width: 200, border: '1px solid #ce9178', padding: 10 }}>
          <h3 style={{ color: '#ce9178' }}>Offset (LIMIT 2 OFFSET 2)</h3>
          {step === 0 && <p>Page 1: [A, B]</p>}
          {step === 1 && <p>User scrolling... item Z added to DB.</p>}
          {step === 2 && (
            <div>
              <p>Page 2: Skip 2, Take 2</p>
              <p style={{ color: 'red' }}>Result: [B, C]</p>
              <p>User sees item B twice!</p>
            </div>
          )}
        </div>

        {/* Cursor */}
        <div style={{ width: 200, border: '1px solid #569cd6', padding: 10 }}>
          <h3 style={{ color: '#569cd6' }}>Cursor (WHERE id < B)</h3>
          {step === 0 && <p>Page 1: [A, B]<br/>NextCursor: B</p>}
          {step === 1 && <p>User scrolling... item Z added to DB.</p>}
          {step === 2 && (
            <div>
              <p>Page 2: Get items after B</p>
              <p style={{ color: '#4EC9B0' }}>Result: [C, D]</p>
              <p>No duplicates!</p>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};
```
