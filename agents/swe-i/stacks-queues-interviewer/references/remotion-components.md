# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## MonotonicStackDemo

```tsx
// Remotion component: MonotonicStackDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const MonotonicStackDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const temps = [73, 74, 75, 71, 69, 72, 76, 73];
  const step = Math.min(Math.floor(frame / (fps * 1.5)), temps.length - 1);

  // Simulate monotonic stack state at each step
  const getStackState = (upToStep: number) => {
    const stack: number[] = [];
    const answers: number[] = new Array(temps.length).fill(0);
    for (let i = 0; i <= upToStep; i++) {
      while (stack.length > 0 && temps[stack[stack.length - 1]] < temps[i]) {
        const prev = stack.pop()!;
        answers[prev] = i - prev;
      }
      stack.push(i);
    }
    return { stack, answers };
  };

  const { stack, answers } = getStackState(step);

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 32, padding: 50 }}>
      <div style={{ marginBottom: 20 }}>Daily Temperatures - Monotonic Stack</div>
      <div style={{ display: 'flex', gap: 12, marginBottom: 30 }}>
        {temps.map((temp, i) => (
          <div
            key={i}
            style={{
              width: 60,
              height: 60,
              border: '3px solid #333',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              background: i === step ? '#FFD700' :
                         stack.includes(i) ? '#FFB6C1' :
                         answers[i] > 0 ? '#90EE90' : 'white',
              borderRadius: 8,
            }}
          >
            {temp}
          </div>
        ))}
      </div>
      <div style={{ marginBottom: 10 }}>
        Stack (indices): [{stack.join(', ')}]
      </div>
      <div>
        Answers: [{answers.join(', ')}]
      </div>
    </div>
  );
};
```
