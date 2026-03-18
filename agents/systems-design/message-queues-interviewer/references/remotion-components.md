# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Exactly-Once / Idempotency Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const IdempotencyDemo = () => {
  const frame = useCurrentFrame();

  // Frame 0-30: Worker gets msg, charges card, crashes
  // Frame 30-60: Timeout, message re-delivered
  // Frame 60-90: Worker gets msg, checks DB, skips charge, ACKs

  const step = Math.floor(frame / 30);

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>At-Least-Once Delivery & Idempotency</h2>

      <div style={{ display: 'flex', justifyContent: 'space-around', marginTop: 30 }}>
        <div style={{ width: 150, border: '1px solid #ce9178', padding: 10 }}>
          <h3>Message Queue</h3>
          <p>Msg ID: XYZ-123</p>
          <p style={{ color: step === 0 ? 'orange' : step === 1 ? 'red' : 'green' }}>
            Status: {step === 0 ? 'In-Flight' : step === 1 ? 'Re-queued' : 'ACKed (Deleted)'}
          </p>
        </div>

        <div style={{ width: 150, border: '1px solid #569cd6', padding: 10 }}>
          <h3>Payment Worker</h3>
          {step === 0 && <p style={{ color: 'red' }}>CRASHES after charge, before ACK!</p>}
          {step === 1 && <p>Restarting...</p>}
          {step === 2 && <p style={{ color: '#4EC9B0' }}>Reads XYZ-123 again.</p>}
        </div>

        <div style={{ width: 150, border: '1px solid #DCDCAA', padding: 10 }}>
          <h3>Database (Idempotency Key)</h3>
          <p>Table: processed_payments</p>
          {step >= 0 && <p>ID: XYZ-123 | Status: SUCCESS</p>}
          {step === 2 && <p style={{ color: 'orange', fontSize: 12 }}>Check: ID exists! Skip Stripe API call.</p>}
        </div>
      </div>
    </div>
  );
};
```
