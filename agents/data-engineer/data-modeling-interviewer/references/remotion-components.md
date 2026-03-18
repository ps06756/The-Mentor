# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of the interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Star Schema Demo

```tsx
// Remotion component: StarSchemaDemo.tsx
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const StarSchemaDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const phaseDuration = 4 * fps; // 4 seconds per phase
  const activePhase = Math.floor(frame / phaseDuration) % 4;
  const progressInPhase = (frame % phaseDuration) / phaseDuration;

  // Phase 0: Show fact table at center
  // Phase 1: Dimensions appear around it (star shape)
  // Phase 2: Highlight a query path (fact -> dim_product -> category)
  // Phase 3: Compare with snowflake schema (normalized dimensions)

  const factOpacity = 1;
  const dimOpacity = activePhase >= 1 ? interpolate(
    activePhase === 1 ? progressInPhase : 1, [0, 0.3], [0, 1], { extrapolateRight: 'clamp' }
  ) : 0;

  const queryHighlight = activePhase === 2;
  const showSnowflake = activePhase === 3;

  const boxStyle = (type: 'fact' | 'dim' | 'highlight' | 'subdim') => ({
    padding: '12px 16px',
    borderRadius: 8,
    border: `2px solid ${
      type === 'fact' ? '#569cd6' :
      type === 'highlight' ? '#4EC9B0' :
      type === 'subdim' ? '#CE9178' :
      '#9CDCFE'
    }`,
    background: type === 'highlight' ? '#2d4a3e' : '#252526',
    color: type === 'fact' ? '#569cd6' : type === 'highlight' ? '#4EC9B0' : '#9CDCFE',
    fontWeight: 'bold' as const,
    fontSize: 13,
    textAlign: 'center' as const,
    minWidth: 120
  });

  const dimensions = [
    { name: 'dim_date', top: 30, left: 340 },
    { name: 'dim_customer', top: 200, left: 600 },
    { name: 'dim_product', top: 370, left: 340 },
    { name: 'dim_store', top: 200, left: 80 }
  ];

  return (
    <div style={{
      width: 800,
      height: 500,
      background: '#1e1e1e',
      color: '#d4d4d4',
      fontFamily: 'monospace',
      padding: 20,
      position: 'relative'
    }}>
      <h2 style={{ color: '#4EC9B0', marginBottom: 10, textAlign: 'center' }}>
        {showSnowflake ? 'Snowflake Schema (Normalized Dimensions)' : 'Star Schema (Denormalized Dimensions)'}
      </h2>

      {!showSnowflake ? (
        <>
          {/* Fact table at center */}
          <div style={{
            position: 'absolute',
            top: 200,
            left: 300,
            opacity: factOpacity
          }}>
            <div style={boxStyle('fact')}>
              fact_sales
              <div style={{ fontSize: 10, color: '#888', marginTop: 4 }}>
                quantity, revenue, discount
              </div>
            </div>
          </div>

          {/* Dimension tables around the fact */}
          {dimensions.map((dim) => (
            <div key={dim.name} style={{
              position: 'absolute',
              top: dim.top,
              left: dim.left,
              opacity: dimOpacity
            }}>
              <div style={boxStyle(queryHighlight && dim.name === 'dim_product' ? 'highlight' : 'dim')}>
                {dim.name}
                <div style={{ fontSize: 10, color: '#888', marginTop: 4 }}>
                  {dim.name === 'dim_product' ? 'name, category, brand' :
                   dim.name === 'dim_date' ? 'month, quarter, year' :
                   dim.name === 'dim_customer' ? 'name, city, segment' :
                   'name, region, manager'}
                </div>
              </div>
            </div>
          ))}

          {/* Query path annotation */}
          {queryHighlight && (
            <div style={{
              position: 'absolute',
              bottom: 30,
              left: 150,
              right: 150,
              padding: 12,
              background: '#2d4a3e',
              borderRadius: 8,
              fontSize: 12,
              textAlign: 'center',
              color: '#4EC9B0'
            }}>
              Query: SELECT category, SUM(revenue) FROM fact_sales JOIN dim_product ... GROUP BY category
              <br />
              Only 1 join needed -- that is the power of a star schema
            </div>
          )}
        </>
      ) : (
        /* Snowflake schema comparison */
        <div style={{ padding: 20 }}>
          <div style={{ display: 'flex', justifyContent: 'center', gap: 20, marginTop: 20 }}>
            <div style={boxStyle('subdim')}>dim_category</div>
            <span style={{ color: '#555', alignSelf: 'center' }}>--&gt;</span>
            <div style={boxStyle('subdim')}>dim_subcategory</div>
            <span style={{ color: '#555', alignSelf: 'center' }}>--&gt;</span>
            <div style={boxStyle('dim')}>dim_product</div>
            <span style={{ color: '#555', alignSelf: 'center' }}>--&gt;</span>
            <div style={boxStyle('fact')}>fact_sales</div>
          </div>

          <div style={{
            marginTop: 40,
            padding: 15,
            background: '#252526',
            borderRadius: 8,
            fontSize: 13,
            lineHeight: 1.8
          }}>
            <strong style={{ color: '#CE9178' }}>Snowflake trade-off:</strong> Less storage redundancy,
            but queries need 3 joins instead of 1 to get category name.
            Most BI tools and analysts prefer the star schema for its simplicity.
            <br /><br />
            <strong style={{ color: '#4EC9B0' }}>Rule of thumb:</strong> Use star schema by default.
            Consider snowflake only when dimension tables are very large
            and storage cost is a real concern.
          </div>
        </div>
      )}
    </div>
  );
};
```
