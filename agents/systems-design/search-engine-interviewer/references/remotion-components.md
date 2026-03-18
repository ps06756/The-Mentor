# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Inverted Index Build & Query Animation

```tsx
// Remotion component: InvertedIndexDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const InvertedIndexDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Animation phases
  // 0-40: Documents appear and get tokenized
  // 40-80: Inverted index builds up term by term
  // 80-120: Query arrives, posting lists are fetched and intersected

  const phase = frame < 40 ? 'tokenize' : frame < 80 ? 'build' : 'query';
  const phaseFrame = phase === 'tokenize' ? frame : phase === 'build' ? frame - 40 : frame - 80;

  const documents = [
    { id: 'D1', text: 'the cat sat on the mat' },
    { id: 'D2', text: 'the dog sat on the log' },
    { id: 'D3', text: 'the cat and the dog' },
  ];

  const indexEntries = [
    { term: 'cat', docs: ['D1', 'D3'] },
    { term: 'sat', docs: ['D1', 'D2'] },
    { term: 'dog', docs: ['D2', 'D3'] },
    { term: 'mat', docs: ['D1'] },
    { term: 'log', docs: ['D2'] },
  ];

  return (
    <div style={{ width: 900, height: 600, background: '#0a0a0a', color: 'white', fontFamily: 'monospace', padding: 20, position: 'relative' }}>
      <h2 style={{ textAlign: 'center', color: '#00bcd4', margin: 0, marginBottom: 20 }}>
        {phase === 'tokenize' ? 'Step 1: Tokenize Documents' :
         phase === 'build' ? 'Step 2: Build Inverted Index' :
         'Step 3: Query "cat sat" -> Intersect'}
      </h2>

      {/* Documents */}
      <div style={{ display: 'flex', gap: 20, marginBottom: 30 }}>
        {documents.map((doc, i) => {
          const visible = phase === 'tokenize' ? phaseFrame > i * 10 : true;
          return (
            <div key={doc.id} style={{
              flex: 1, padding: 12, background: '#1a1a2e', borderRadius: 8,
              border: '1px solid #333', opacity: visible ? 1 : 0
            }}>
              <div style={{ color: '#00bcd4', marginBottom: 6 }}>{doc.id}</div>
              <div style={{ fontSize: 12, color: '#ccc' }}>{doc.text}</div>
            </div>
          );
        })}
      </div>

      {/* Inverted Index */}
      {(phase === 'build' || phase === 'query') && (
        <div style={{ background: '#1a1a2e', borderRadius: 8, padding: 16, border: '1px solid #333' }}>
          <div style={{ color: '#ff9800', marginBottom: 12 }}>Inverted Index:</div>
          {indexEntries.map((entry, i) => {
            const visible = phase === 'build' ? phaseFrame > i * 7 : true;
            const highlighted = phase === 'query' && (entry.term === 'cat' || entry.term === 'sat');
            return (
              <div key={entry.term} style={{
                display: 'flex', gap: 20, padding: '4px 0',
                opacity: visible ? 1 : 0,
                color: highlighted ? '#4caf50' : '#ccc',
                fontWeight: highlighted ? 'bold' : 'normal'
              }}>
                <span style={{ width: 60 }}>{entry.term}</span>
                <span>-&gt; [{entry.docs.join(', ')}]</span>
              </div>
            );
          })}
        </div>
      )}

      {/* Query result */}
      {phase === 'query' && phaseFrame > 20 && (
        <div style={{
          position: 'absolute', bottom: 40, left: '50%', transform: 'translateX(-50%)',
          padding: '12px 24px', background: '#4caf50', borderRadius: 8, fontSize: 18
        }}>
          Intersection: D1 (cat AND sat both appear)
        </div>
      )}
    </div>
  );
};
```
