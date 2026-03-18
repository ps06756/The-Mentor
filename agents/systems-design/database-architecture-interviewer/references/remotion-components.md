# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Database Replication Lag Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const ReplicationLagDemo = () => {
  const frame = useCurrentFrame();

  // Animation logic: User writes profile update, immediately reads from replica
  const writeOccurred = frame > 20;
  const readOccurred = frame > 40;
  const replicationComplete = frame > 70;

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Replication Lag Issue</h2>

      <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: 50 }}>
        {/* Primary DB */}
        <div style={{ width: 200, textAlign: 'center', border: '2px solid #569CD6', padding: 10 }}>
          <h3>Primary DB</h3>
          <p>Name: {writeOccurred ? "Alice_New" : "Alice_Old"}</p>
          <div style={{ color: writeOccurred && frame < 30 ? '#4EC9B0' : 'transparent' }}>Write acknowledged!</div>
        </div>

        {/* Network/Replication Line */}
        <div style={{ flex: 1, position: 'relative' }}>
          <div style={{
            position: 'absolute', top: '50%', left: 0, right: 0, height: 2, background: '#555'
          }} />
          {writeOccurred && !replicationComplete && (
            <div style={{
              position: 'absolute', top: '50%', left: `${(frame - 20) * 2}%`,
              transform: 'translateY(-50%)', width: 10, height: 10, background: 'orange', borderRadius: '50%'
            }} />
          )}
        </div>

        {/* Replica DB */}
        <div style={{ width: 200, textAlign: 'center', border: '2px solid #CE9178', padding: 10 }}>
          <h3>Read Replica</h3>
          <p>Name: {replicationComplete ? "Alice_New" : "Alice_Old"}</p>
        </div>
      </div>

      {/* Client View */}
      <div style={{ marginTop: 50, textAlign: 'center' }}>
        <h3>Client Application</h3>
        {readOccurred && !replicationComplete && (
          <p style={{ color: 'red' }}>Reads from Replica: "Alice_Old" (Stale Data!)</p>
        )}
        {replicationComplete && (
          <p style={{ color: 'green' }}>Reads from Replica: "Alice_New" (Consistent)</p>
        )}
      </div>
    </div>
  );
};
```
