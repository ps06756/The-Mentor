# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of the interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## SCD Type 2 Animation

```tsx
// Remotion component: SCDType2Visualizer.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const SCDType2Visualizer = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const scenarioDuration = 5 * fps;
  const activePhase = Math.floor(frame / scenarioDuration);
  const progress = (frame % scenarioDuration) / scenarioDuration;

  const phases = [
    { name: 'Initial State', description: 'Customer John in Boston' },
    { name: 'Change Detected', description: 'John moves to Chicago' },
    { name: 'SCD Type 2 Applied', description: 'Close old record, create new' },
    { name: 'Query History', description: 'Can answer "Where was John in May?"' }
  ];

  const currentPhase = phases[Math.min(activePhase, phases.length - 1)];

  return (
    <div style={{
      width: 900,
      height: 600,
      background: '#1e1e1e',
      color: '#d4d4d4',
      fontFamily: 'monospace',
      padding: 40
    }}>
      <h2 style={{ color: '#4EC9B0', marginBottom: 20 }}>SCD Type 2: Tracking History</h2>

      <div style={{ marginBottom: 30 }}>
        <strong style={{ color: '#9CDCFE' }}>Phase: </strong>
        <span style={{ color: '#CE9178', fontSize: 18 }}>{currentPhase.name}</span>
        <p style={{ color: '#6A9955', marginTop: 10 }}>{currentPhase.description}</p>
      </div>

      {/* Customer Dimension Table */}
      <div style={{
        background: '#252526',
        padding: 20,
        borderRadius: 8,
        marginBottom: 20
      }}>
        <h3 style={{ color: '#569cd6', marginBottom: 15 }}>customer_dimension</h3>

        {/* Table Header */}
        <div style={{ display: 'flex', borderBottom: '2px solid #555', paddingBottom: 10, marginBottom: 10 }}>
          {['cust_sk', 'natural_id', 'name', 'city', 'start_date', 'end_date', 'is_current'].map(col => (
            <div key={col} style={{ width: 110, color: '#9CDCFE', fontSize: 12 }}>{col}</div>
          ))}
        </div>

        {/* Row 1: Original Record */}
        <div style={{
          display: 'flex',
          padding: '8px 0',
          background: activePhase >= 1 ? '#3c3c3c' : 'transparent',
          opacity: activePhase >= 1 ? 0.7 : 1
        }}>
          <div style={{ width: 110 }}>101</div>
          <div style={{ width: 110 }}>JD-001</div>
          <div style={{ width: 110 }}>John Doe</div>
          <div style={{ width: 110, color: activePhase >= 1 ? '#CE9178' : '#d4d4d4' }}>
            {activePhase >= 1 ? 'Boston ✗' : 'Boston'}
          </div>
          <div style={{ width: 110 }}>2023-01-01</div>
          <div style={{ width: 110, color: activePhase >= 2 ? '#CE9178' : '#d4d4d4' }}>
            {activePhase >= 2 ? '2023-06-15' : '9999-12-31'}
          </div>
          <div style={{ width: 110, color: activePhase >= 2 ? '#F44747' : '#4EC9B0' }}>
            {activePhase >= 2 ? 'N' : 'Y'}
          </div>
        </div>

        {/* Row 2: New Record (appears in phase 2) */}
        {activePhase >= 2 && (
          <div style={{
            display: 'flex',
            padding: '8px 0',
            background: '#1e3a1e',
            animation: 'fadeIn 0.5s'
          }}>
            <div style={{ width: 110, color: '#4EC9B0' }}>102</div>
            <div style={{ width: 110 }}>JD-001</div>
            <div style={{ width: 110 }}>John Doe</div>
            <div style={{ width: 110, color: '#4EC9B0' }}>Chicago ✓</div>
            <div style={{ width: 110, color: '#4EC9B0' }}>2023-06-15</div>
            <div style={{ width: 110 }}>9999-12-31</div>
            <div style={{ width: 110, color: '#4EC9B0' }}>Y</div>
          </div>
        )}
      </div>

      {/* Query Example */}
      {activePhase >= 3 && (
        <div style={{
          background: '#2d2d2d',
          padding: 15,
          borderRadius: 8,
          border: '1px solid #4EC9B0'
        }}>
          <strong style={{ color: '#4EC9B0' }}>Query Example:</strong>
          <pre style={{ marginTop: 10, color: '#d4d4d4' }}>
{`-- Where did John live in May 2023?
SELECT city FROM customer_dimension
WHERE natural_id = 'JD-001'
  AND '2023-05-01' BETWEEN start_date AND end_date;

-- Result: Boston (correct!)`}
          </pre>
        </div>
      )}

      <div style={{ marginTop: 20, fontSize: 12, color: '#858585' }}>
        Facts always join to the surrogate key (cust_sk), not the natural key
      </div>
    </div>
  );
};
```
