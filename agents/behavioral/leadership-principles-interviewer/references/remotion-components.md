# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## STAR Method Breakdown Animation

```tsx
// Remotion component: STARMethodDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const STARMethodDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Animation phases: each STAR component appears in sequence
  // 0-30: Situation
  // 30-60: Task
  // 60-100: Action (longest, as it should be the bulk of the answer)
  // 100-130: Result

  const steps = [
    { label: 'S - Situation', color: '#3498db', start: 0, description: 'Set the scene. When, where, what context?' },
    { label: 'T - Task', color: '#e67e22', start: 30, description: 'What was YOUR specific responsibility?' },
    { label: 'A - Action', color: '#2ecc71', start: 60, description: 'What did YOU do? This is the longest part.' },
    { label: 'R - Result', color: '#9b59b6', start: 100, description: 'What was the measurable outcome? What did you learn?' },
  ];

  return (
    <div style={{ width: 700, height: 500, background: '#0a0a0a', color: 'white', fontFamily: 'sans-serif', padding: 30, position: 'relative' }}>
      <h2 style={{ textAlign: 'center', marginBottom: 30, color: '#ecf0f1' }}>
        The STAR Method
      </h2>

      {steps.map((step, i) => {
        const progress = Math.min(1, Math.max(0, (frame - step.start) / 20));
        const isActive = frame >= step.start && (i === steps.length - 1 || frame < steps[i + 1].start);

        return (
          <div key={step.label} style={{
            display: 'flex', alignItems: 'center', gap: 20, marginBottom: 20,
            opacity: progress,
            transform: `translateX(${(1 - progress) * 50}px)`,
          }}>
            {/* Circle indicator */}
            <div style={{
              width: 50, height: 50, borderRadius: '50%',
              background: isActive ? step.color : '#333',
              display: 'flex', alignItems: 'center', justifyContent: 'center',
              fontSize: 24, fontWeight: 'bold',
              transition: 'background 0.3s',
              flexShrink: 0,
            }}>
              {step.label[0]}
            </div>

            {/* Label and description */}
            <div>
              <div style={{ fontSize: 18, fontWeight: 'bold', color: step.color }}>
                {step.label}
              </div>
              <div style={{ fontSize: 14, color: '#999', marginTop: 4 }}>
                {step.description}
              </div>
            </div>
          </div>
        );
      })}

      {/* Tip at bottom */}
      {frame > 110 && (
        <div style={{
          position: 'absolute', bottom: 30, left: 30, right: 30,
          padding: 16, background: '#1a1a2e', borderRadius: 8,
          borderLeft: '4px solid #e74c3c', fontSize: 14, color: '#ccc'
        }}>
          Tip: The Action section should be 50-60% of your answer. Use "I" not "we."
        </div>
      )}
    </div>
  );
};
```
