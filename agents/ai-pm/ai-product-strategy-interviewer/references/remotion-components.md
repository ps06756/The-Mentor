# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of AI product strategy concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## AI Product Decision Tree Animation

```tsx
// Remotion component: AIProductDecisionTreeDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const AIProductDecisionTreeDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Animation phases: each decision node appears in sequence
  // 0-40: "Can you define rules?" node
  // 40-80: "Do you have data?" node
  // 80-120: "Cost of being wrong?" node
  // 120-160: "Economics check" node

  const nodes = [
    { label: 'Can you define explicit rules?', yes: 'Use rules engine', no: 'Continue', color: '#3498db', start: 0 },
    { label: 'Do you have labeled data?', yes: 'Train a model', no: 'Try LLM zero-shot', color: '#e67e22', start: 40 },
    { label: 'Cost of being wrong?', yes: 'Human-in-the-loop', no: 'Ship with thresholds', color: '#e74c3c', start: 80 },
    { label: 'Unit economics work?', yes: 'Build it (MVP + eval)', no: 'Rethink approach', color: '#2ecc71', start: 120 },
  ];

  return (
    <div style={{ width: 700, height: 500, background: '#0a0a0a', color: 'white', fontFamily: 'sans-serif', padding: 30, position: 'relative' }}>
      <h2 style={{ textAlign: 'center', marginBottom: 30, color: '#ecf0f1' }}>
        AI vs Non-AI Decision Framework
      </h2>

      {nodes.map((node, i) => {
        const progress = Math.min(1, Math.max(0, (frame - node.start) / 25));
        const isActive = frame >= node.start && (i === nodes.length - 1 || frame < nodes[i + 1].start);

        return (
          <div key={node.label} style={{
            display: 'flex', alignItems: 'center', gap: 16, marginBottom: 16,
            opacity: progress,
            transform: `translateY(${(1 - progress) * 30}px)`,
          }}>
            <div style={{
              width: 44, height: 44, borderRadius: 8,
              background: isActive ? node.color : '#333',
              display: 'flex', alignItems: 'center', justifyContent: 'center',
              fontSize: 18, fontWeight: 'bold', flexShrink: 0,
            }}>
              {i + 1}
            </div>
            <div>
              <div style={{ fontSize: 16, fontWeight: 'bold', color: node.color }}>{node.label}</div>
              <div style={{ fontSize: 13, color: '#888', marginTop: 2 }}>
                YES: {node.yes} | NO: {node.no}
              </div>
            </div>
          </div>
        );
      })}

      {frame > 140 && (
        <div style={{
          position: 'absolute', bottom: 30, left: 30, right: 30,
          padding: 14, background: '#1a1a2e', borderRadius: 8,
          borderLeft: '4px solid #f39c12', fontSize: 13, color: '#ccc'
        }}>
          Key principle: AI should be the last resort, not the first instinct. Start simple, add intelligence where it creates measurable value.
        </div>
      )}
    </div>
  );
};
```
