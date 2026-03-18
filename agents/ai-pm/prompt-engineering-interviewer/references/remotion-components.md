# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of prompt engineering and RAG concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## RAG Pipeline Animation

```tsx
// Remotion component: RAGPipelineDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const RAGPipelineDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Animation phases: each pipeline stage appears in sequence
  // 0-30: Query Processing
  // 30-60: Retrieval
  // 60-90: Context Assembly
  // 90-120: Generation
  // 120-150: Post-Processing

  const stages = [
    { label: 'Query Processing', detail: 'Rewrite, classify, embed', color: '#3498db', start: 0, icon: 'Q' },
    { label: 'Retrieval', detail: 'Vector + keyword search, rerank', color: '#e67e22', start: 30, icon: 'R' },
    { label: 'Context Assembly', detail: 'Select top-N, order, fit token budget', color: '#2ecc71', start: 60, icon: 'C' },
    { label: 'Generation', detail: 'System prompt + context + query -> LLM', color: '#9b59b6', start: 90, icon: 'G' },
    { label: 'Post-Processing', detail: 'Verify citations, detect hallucinations', color: '#e74c3c', start: 120, icon: 'P' },
  ];

  return (
    <div style={{ width: 700, height: 500, background: '#0a0a0a', color: 'white', fontFamily: 'sans-serif', padding: 30, position: 'relative' }}>
      <h2 style={{ textAlign: 'center', marginBottom: 30, color: '#ecf0f1' }}>
        RAG Pipeline Architecture
      </h2>

      {stages.map((stage, i) => {
        const progress = Math.min(1, Math.max(0, (frame - stage.start) / 20));
        const isActive = frame >= stage.start && (i === stages.length - 1 || frame < stages[i + 1].start);

        return (
          <div key={stage.label} style={{
            display: 'flex', alignItems: 'center', gap: 16, marginBottom: 14,
            opacity: progress,
            transform: `translateX(${(1 - progress) * 40}px)`,
          }}>
            <div style={{
              width: 44, height: 44, borderRadius: '50%',
              background: isActive ? stage.color : '#333',
              display: 'flex', alignItems: 'center', justifyContent: 'center',
              fontSize: 18, fontWeight: 'bold', flexShrink: 0,
            }}>
              {stage.icon}
            </div>
            <div>
              <div style={{ fontSize: 16, fontWeight: 'bold', color: stage.color }}>{stage.label}</div>
              <div style={{ fontSize: 13, color: '#888', marginTop: 2 }}>{stage.detail}</div>
            </div>
          </div>
        );
      })}

      {frame > 130 && (
        <div style={{
          position: 'absolute', bottom: 30, left: 30, right: 30,
          padding: 14, background: '#1a1a2e', borderRadius: 8,
          borderLeft: '4px solid #e74c3c', fontSize: 13, color: '#ccc'
        }}>
          Key insight: RAG quality is bottlenecked by retrieval, not generation. Fix retrieval first.
        </div>
      )}
    </div>
  );
};
```
