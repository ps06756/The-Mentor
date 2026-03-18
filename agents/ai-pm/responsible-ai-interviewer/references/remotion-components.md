# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of responsible AI and bias detection concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Bias Detection Pipeline Animation

```tsx
// Remotion component: BiasDetectionDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const BiasDetectionDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Animation phases: each detection stage appears in sequence
  // 0-35: Data Audit
  // 35-70: Model Evaluation
  // 70-105: Disparate Impact Analysis
  // 105-140: Mitigation & Monitoring

  const stages = [
    { label: 'Data Audit', detail: 'Check training data demographics and representation', color: '#3498db', start: 0, icon: 'D' },
    { label: 'Model Evaluation', detail: 'Stratified metrics across protected groups', color: '#e67e22', start: 35, icon: 'E' },
    { label: 'Disparate Impact', detail: 'Measure outcome ratios (4/5ths rule, demographic parity)', color: '#e74c3c', start: 70, icon: 'I' },
    { label: 'Mitigate & Monitor', detail: 'Fairness constraints, ongoing audits, transparency', color: '#2ecc71', start: 105, icon: 'M' },
  ];

  return (
    <div style={{ width: 700, height: 500, background: '#0a0a0a', color: 'white', fontFamily: 'sans-serif', padding: 30, position: 'relative' }}>
      <h2 style={{ textAlign: 'center', marginBottom: 30, color: '#ecf0f1' }}>
        Bias Detection Pipeline
      </h2>

      {stages.map((stage, i) => {
        const progress = Math.min(1, Math.max(0, (frame - stage.start) / 22));
        const isActive = frame >= stage.start && (i === stages.length - 1 || frame < stages[i + 1].start);

        return (
          <div key={stage.label} style={{
            display: 'flex', alignItems: 'center', gap: 16, marginBottom: 20,
            opacity: progress,
            transform: `translateX(${(1 - progress) * 50}px)`,
          }}>
            <div style={{
              width: 48, height: 48, borderRadius: '50%',
              background: isActive ? stage.color : '#333',
              display: 'flex', alignItems: 'center', justifyContent: 'center',
              fontSize: 20, fontWeight: 'bold', flexShrink: 0,
            }}>
              {stage.icon}
            </div>
            <div>
              <div style={{ fontSize: 16, fontWeight: 'bold', color: stage.color }}>{stage.label}</div>
              <div style={{ fontSize: 13, color: '#888', marginTop: 3 }}>{stage.detail}</div>
            </div>
          </div>
        );
      })}

      {frame > 120 && (
        <div style={{
          position: 'absolute', bottom: 30, left: 30, right: 30,
          padding: 14, background: '#1a1a2e', borderRadius: 8,
          borderLeft: '4px solid #e74c3c', fontSize: 13, color: '#ccc'
        }}>
          Key principle: Bias is not a bug you fix once. It is a property you measure and manage continuously.
        </div>
      )}
    </div>
  );
};
```
