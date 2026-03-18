# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Consistent Hashing vs Modulo Hashing Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const LoadBalancingHashDemo = () => {
  const frame = useCurrentFrame();

  // Phase 1 (Frame 0-30): Modulo Hashing (N=3)
  // Phase 2 (Frame 30-60): Server Dies (N=2) -> Massive reshuffle
  // Phase 3 (Frame 60-90): Consistent Hashing Ring

  const step = Math.floor(frame / 30);

  return (
    <div style={{ width: 600, height: 400, background: '#1e1e1e', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Traditional Modulo vs Consistent Hashing</h2>

      {step < 2 ? (
        <div style={{ marginTop: 20 }}>
          <h3 style={{ color: '#ce9178', textAlign: 'center' }}>Modulo Hashing: Hash(IP) % N</h3>
          <div style={{ display: 'flex', justifyContent: 'space-around', marginTop: 20 }}>
            <div style={{ border: '1px solid #555', padding: 10 }}>
              <h4>Server 0</h4>
              <p>User A (Hash=0)</p>
              <p style={{ color: step === 1 ? 'red' : 'white' }}>User D (Hash=3)</p>
            </div>
            <div style={{ border: '1px solid #555', padding: 10, background: step === 1 ? '#440000' : 'transparent' }}>
              <h4>{step === 1 ? 'Server 1 (DEAD)' : 'Server 1'}</h4>
              {step === 0 && <><p>User B (Hash=1)</p><p>User E (Hash=4)</p></>}
            </div>
            <div style={{ border: '1px solid #555', padding: 10 }}>
              <h4>Server 2</h4>
              <p style={{ color: step === 1 ? 'red' : 'white' }}>User C (Hash=2)</p>
            </div>
          </div>
          {step === 1 && (
            <div style={{ textAlign: 'center', marginTop: 20, color: 'orange' }}>
              N drops from 3 to 2. <br/>
              User D (3 % 2 = 1) moves to Server 1 (Dead). User C (2 % 2 = 0) moves to Server 0.<br/>
              Almost ALL users are mapped to a different server. Massive cache misses!
            </div>
          )}
        </div>
      ) : (
        <div style={{ marginTop: 20 }}>
          <h3 style={{ color: '#4EC9B0', textAlign: 'center' }}>Consistent Hashing (Ring)</h3>
          <div style={{ position: 'relative', width: 200, height: 200, margin: '0 auto', borderRadius: '50%', border: '2px solid #555' }}>
            {/* Servers */}
            <div style={{ position: 'absolute', top: -15, left: 85, background: '#569cd6', padding: '2px 5px', borderRadius: 4 }}>S0</div>
            <div style={{ position: 'absolute', bottom: 20, right: -15, background: '#569cd6', padding: '2px 5px', borderRadius: 4 }}>S1</div>
            <div style={{ position: 'absolute', bottom: 20, left: -15, background: '#569cd6', padding: '2px 5px', borderRadius: 4 }}>S2</div>

            {/* Users mapping clockwise */}
            <div style={{ position: 'absolute', top: 20, right: 20, color: 'orange' }}>User A -> S1</div>
          </div>
          <div style={{ textAlign: 'center', marginTop: 20, color: '#DCDCAA' }}>
            If S1 dies, User A maps to S2. Users already mapped to S0 and S2 DO NOT MOVE.<br/>
            Only 1/N keys are remapped.
          </div>
        </div>
      )}
    </div>
  );
};
```
