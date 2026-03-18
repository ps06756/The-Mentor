# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of the interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Backpressure Handling Animation

```tsx
// Remotion component: BackpressureVisualizer.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const BackpressureVisualizer = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const scenarioDuration = 4 * fps; // 4 seconds per scenario
  const activeScenario = Math.floor(frame / scenarioDuration) % 3;
  const progressInScenario = (frame % scenarioDuration) / scenarioDuration;

  // Scenario 0: Healthy (Producer 100/sec, Consumer 100/sec)
  // Scenario 1: Mild backpressure (Producer 200/sec, Consumer 100/sec)
  // Scenario 2: Severe backpressure (Producer 1000/sec, Consumer 100/sec)

  const scenarios = [
    { name: 'Healthy Flow', producerRate: 100, consumerRate: 100, queueMax: 100 },
    { name: 'Mild Backpressure', producerRate: 200, consumerRate: 100, queueMax: 500 },
    { name: 'Severe Backpressure', producerRate: 1000, consumerRate: 100, queueMax: 1000 }
  ];

  const current = scenarios[activeScenario];
  const queueSize = Math.min(
    Math.floor(progressInScenario * (current.producerRate - current.consumerRate) * 10),
    current.queueMax
  );

  const isHealthy = queueSize < 100;
  const isWarning = queueSize >= 100 && queueSize < 500;
  const isCritical = queueSize >= 500;

  return (
    <div style={{
      width: 800,
      height: 500,
      background: '#1e1e1e',
      color: '#d4d4d4',
      fontFamily: 'monospace',
      padding: 40
    }}>
      <h2 style={{ color: '#4EC9B0', marginBottom: 30 }}>Backpressure Handling</h2>

      <div style={{ marginBottom: 20 }}>
        <strong style={{ color: '#9CDCFE' }}>Scenario: </strong>
        <span style={{
          color: isCritical ? '#F44747' : isWarning ? '#CE9178' : '#4EC9B0',
          fontSize: 18
        }}>
          {current.name}
        </span>
      </div>

      {/* Producer */}
      <div style={{ display: 'flex', alignItems: 'center', marginBottom: 20 }}>
        <div style={{ width: 120, color: '#569cd6' }}>Producer:</div>
        <div style={{
          width: current.producerRate,
          height: 30,
          background: '#569cd6',
          borderRadius: 4
        }} />
        <span style={{ marginLeft: 10 }}>{current.producerRate} events/sec</span>
      </div>

      {/* Queue */}
      <div style={{ display: 'flex', alignItems: 'center', marginBottom: 20 }}>
        <div style={{ width: 120, color: '#9CDCFE' }}>Queue:</div>
        <div style={{
          width: 600,
          height: 40,
          border: '2px solid #555',
          borderRadius: 4,
          position: 'relative',
          overflow: 'hidden'
        }}>
          <div style={{
            position: 'absolute',
            left: 0,
            top: 0,
            height: '100%',
            width: `${(queueSize / current.queueMax) * 100}%`,
            background: isCritical ? '#F44747' : isWarning ? '#CE9178' : '#4EC9B0',
            transition: 'width 0.1s'
          }} />
          <span style={{
            position: 'absolute',
            left: 10,
            top: 10,
            color: '#fff',
            fontWeight: 'bold'
          }}>
            {queueSize} messages
          </span>
        </div>
      </div>

      {/* Consumer */}
      <div style={{ display: 'flex', alignItems: 'center', marginBottom: 30 }}>
        <div style={{ width: 120, color: '#C586C0' }}>Consumer:</div>
        <div style={{
          width: current.consumerRate,
          height: 30,
          background: '#C586C0',
          borderRadius: 4
        }} />
        <span style={{ marginLeft: 10 }}>{current.consumerRate} events/sec</span>
      </div>

      {/* Strategies */}
      <div style={{
        padding: 15,
        background: '#252526',
        borderRadius: 8,
        fontSize: 14
      }}>
        <strong>Backpressure Strategies:</strong>
        <ul style={{ marginTop: 10, lineHeight: 1.8 }}>
          <li>✓ <strong>Drop oldest:</strong> When queue full, discard old messages</li>
          <li>✓ <strong>Throttle producer:</strong> Block/slow down upstream</li>
          <li>✓ <strong>Scale consumers:</strong> Add more processing capacity</li>
          <li>✓ <strong>Dead letter queue:</strong> Route failed messages separately</li>
        </ul>
      </div>
    </div>
  );
};
```
