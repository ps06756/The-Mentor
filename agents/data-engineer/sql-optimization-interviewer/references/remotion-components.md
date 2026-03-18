# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of the interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Query Plan Animation

```tsx
// Remotion component: QueryPlanVisualizer.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const QueryPlanVisualizer = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const nodes = [
    { id: 1, label: 'Seq Scan on orders', cost: 'cost=0.00..18406.00', rows: 1000000 },
    { id: 2, label: 'Filter: (amount > 1000)', cost: 'cost=18406.00..18406.50', rows: 5000 },
    { id: 3, label: 'Sort by created_at', cost: 'cost=18406.50..18419.00', rows: 5000 },
  ];

  const activeNode = Math.floor(frame / (2 * fps)) % nodes.length;

  return (
    <div style={{ fontFamily: 'monospace', padding: 40, background: '#1e1e1e', color: '#d4d4d4' }}>
      <h2 style={{ color: '#4EC9B0' }}>Query Execution Plan</h2>
      {nodes.map((node, idx) => (
        <div
          key={node.id}
          style={{
            margin: '10px 0',
            padding: 15,
            border: '2px solid',
            borderColor: idx === activeNode ? '#4EC9B0' : '#555',
            background: idx === activeNode ? '#264f4f' : '#2d2d2d',
            opacity: idx <= activeNode ? 1 : 0.3,
            transition: 'all 0.3s'
          }}
        >
          <div style={{ fontWeight: 'bold', color: '#9CDCFE' }}>→ {node.label}</div>
          <div style={{ fontSize: 14, color: '#CE9178', marginTop: 5 }}>
            Cost: {node.cost} | Rows: {node.rows.toLocaleString()}
          </div>
        </div>
      ))}
    </div>
  );
};
```
