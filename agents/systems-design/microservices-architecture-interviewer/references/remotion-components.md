# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## The Saga Pattern (Choreography) Demo

```tsx
import { useCurrentFrame } from 'remotion';

export const SagaPatternDemo = () => {
  const frame = useCurrentFrame();

  // Animation phases
  const orderCreated = frame > 20;
  const inventoryReserved = frame > 50;
  const paymentProcessed = frame > 80;

  return (
    <div style={{ width: 600, height: 400, background: '#2d2d2d', color: 'white', padding: 20 }}>
      <h2 style={{ textAlign: 'center' }}>Event-Driven Saga (Choreography)</h2>

      <div style={{ display: 'flex', justifyContent: 'space-around', marginTop: 50 }}>

        <div style={{ textAlign: 'center', opacity: orderCreated ? 1 : 0.3 }}>
          <div style={{ padding: 10, background: '#569CD6', borderRadius: 8 }}>Order Service</div>
          <div style={{ fontSize: 12, marginTop: 10 }}>1. Create Order (PENDING)</div>
          {orderCreated && <div style={{ color: '#4EC9B0', marginTop: 5 }}>Pub: OrderCreated</div>}
        </div>

        <div style={{ textAlign: 'center', opacity: inventoryReserved ? 1 : 0.3 }}>
          <div style={{ padding: 10, background: '#CE9178', borderRadius: 8 }}>Inventory Service</div>
          <div style={{ fontSize: 12, marginTop: 10 }}>2. Reserve Items</div>
          {inventoryReserved && <div style={{ color: '#4EC9B0', marginTop: 5 }}>Pub: InventoryReserved</div>}
        </div>

        <div style={{ textAlign: 'center', opacity: paymentProcessed ? 1 : 0.3 }}>
          <div style={{ padding: 10, background: '#DCDCAA', borderRadius: 8, color: 'black' }}>Payment Service</div>
          <div style={{ fontSize: 12, marginTop: 10 }}>3. Charge Card</div>
          {paymentProcessed && <div style={{ color: '#4EC9B0', marginTop: 5 }}>Pub: PaymentSuccess</div>}
        </div>

      </div>

      <div style={{ marginTop: 80, textAlign: 'center', padding: 10, borderTop: '2px dashed #666' }}>
        <h3>Event Bus (Kafka)</h3>
        <p style={{ fontFamily: 'monospace', color: '#ccc' }}>
          {orderCreated && !inventoryReserved ? "> [OrderCreated Event]" : ""}
          {inventoryReserved && !paymentProcessed ? "> [InventoryReserved Event]" : ""}
          {paymentProcessed ? "> [PaymentSuccess Event] -> Order status: APPROVED" : ""}
        </p>
      </div>
    </div>
  );
};
```
