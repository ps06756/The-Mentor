# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Binary Tree Traversal Demo

```tsx
// Remotion component: BinaryTreeTraversalDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const BinaryTreeTraversalDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const nodes = [
    { val: 1, x: 300, y: 50, left: 2, right: 3 },
    { val: 2, x: 150, y: 150, left: 4, right: 5 },
    { val: 3, x: 450, y: 150, left: null, right: null },
    { val: 4, x: 75, y: 250, left: null, right: null },
    { val: 5, x: 225, y: 250, left: null, right: null },
  ];
  const inorderSeq = [4, 2, 5, 1, 3];
  const step = Math.min(Math.floor(frame / (fps * 0.8)), inorderSeq.length - 1);

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 24, padding: 20 }}>
      <div>Inorder: {inorderSeq.slice(0, step + 1).join(', ')}</div>
      <svg width={600} height={300}>
        {nodes.map((n) => (
          <circle key={n.val} cx={n.x} cy={n.y} r={25}
            fill={n.val === inorderSeq[step] ? '#90EE90' : 'white'}
            stroke="#333" strokeWidth={2} />
        ))}
      </svg>
    </div>
  );
};
```
