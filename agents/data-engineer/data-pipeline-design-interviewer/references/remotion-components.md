# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of the interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## ETL Pipeline Flow Demo

```tsx
// Remotion component: ETLPipelineFlowDemo.tsx
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const ETLPipelineFlowDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const phaseDuration = 3 * fps; // 3 seconds per phase
  const activePhase = Math.floor(frame / phaseDuration) % 3;
  const progressInPhase = (frame % phaseDuration) / phaseDuration;

  // Phase 0: ETL flow (transform before load)
  // Phase 1: ELT flow (load before transform)
  // Phase 2: Side-by-side comparison

  const phases = [
    { name: 'ETL: Extract -> Transform -> Load', style: 'etl' },
    { name: 'ELT: Extract -> Load -> Transform', style: 'elt' },
    { name: 'Comparison: When to Use Each', style: 'compare' }
  ];

  const current = phases[activePhase];

  const dataPosition = interpolate(progressInPhase, [0, 1], [0, 600], {
    extrapolateRight: 'clamp'
  });

  const boxStyle = (highlight: boolean) => ({
    width: 150,
    height: 80,
    border: `2px solid ${highlight ? '#4EC9B0' : '#555'}`,
    borderRadius: 8,
    display: 'flex' as const,
    alignItems: 'center' as const,
    justifyContent: 'center' as const,
    background: highlight ? '#2d4a3e' : '#252526',
    color: highlight ? '#4EC9B0' : '#9CDCFE',
    fontWeight: 'bold' as const,
    fontSize: 14
  });

  const getHighlightIndex = () => {
    if (progressInPhase < 0.33) return 0;
    if (progressInPhase < 0.66) return 1;
    return 2;
  };

  const highlightIdx = getHighlightIndex();

  const etlStages = ['Extract', 'Transform', 'Load'];
  const eltStages = ['Extract', 'Load', 'Transform'];
  const stages = current.style === 'elt' ? eltStages : etlStages;

  return (
    <div style={{
      width: 800,
      height: 500,
      background: '#1e1e1e',
      color: '#d4d4d4',
      fontFamily: 'monospace',
      padding: 40
    }}>
      <h2 style={{ color: '#4EC9B0', marginBottom: 20 }}>
        {current.name}
      </h2>

      {current.style !== 'compare' ? (
        <div style={{ display: 'flex', alignItems: 'center', gap: 30, marginTop: 40 }}>
          {stages.map((stage, i) => (
            <div key={stage} style={{ display: 'flex', alignItems: 'center', gap: 15 }}>
              <div style={boxStyle(i === highlightIdx)}>
                {stage}
              </div>
              {i < stages.length - 1 && (
                <span style={{ color: '#569cd6', fontSize: 24 }}>--></span>
              )}
            </div>
          ))}
        </div>
      ) : (
        <div style={{ marginTop: 30 }}>
          <div style={{ marginBottom: 20, padding: 15, background: '#252526', borderRadius: 8 }}>
            <strong style={{ color: '#569cd6' }}>ETL:</strong>
            <span> Legacy systems, sensitive data filtering, limited warehouse compute</span>
          </div>
          <div style={{ padding: 15, background: '#252526', borderRadius: 8 }}>
            <strong style={{ color: '#C586C0' }}>ELT:</strong>
            <span> Cloud warehouses, raw data preservation, flexible re-transformation</span>
          </div>
        </div>
      )}

      <div style={{
        marginTop: 60,
        padding: 15,
        background: '#252526',
        borderRadius: 8,
        fontSize: 13,
        lineHeight: 1.8
      }}>
        <strong>Key Insight:</strong> Modern cloud warehouses (Snowflake, BigQuery) have made
        ELT the default choice for most analytics pipelines. ETL is still preferred when you
        need to filter sensitive data before it reaches the warehouse, or when transformation
        requires compute that the warehouse cannot efficiently provide.
      </div>
    </div>
  );
};
```
