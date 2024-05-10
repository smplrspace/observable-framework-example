---
toc: false
---

# Smplrspace on Observable

```js
import { sensors } from "./data/sensors.js";
import { co2Data } from "./data/co2.js";
import { pipe, A, D } from "@mobily/ts-belt";
```

```js
display(co2Data);
```

```js
const sensorDataClean = co2Data.map((d) => ({
  ...d,
  dateNew: d3.utcParse("%Y-%m-%dT%H:%M:%SZ")(d.ts),
}));
```

```js
const firstTenUuids = pipe(
  sensorDataClean,
  A.map(D.get("uuid")),
  A.uniq,
  A.take(10)
);
const limitedSensorDataClean = pipe(
  sensorDataClean,
  A.filter((d) => A.includes(firstTenUuids, d.uuid))
);
```

```js
const testChart = Plot.plot({
  color: { legend: true },
  marks: [
    Plot.areaY(limitedSensorDataClean, {
      x: "dateNew",
      y: "value",
      fill: "uuid",
      tip: true,
    }),
  ],
});
```

```js
display(testChart);
```

```js
display(sensorDataClean);
```

```js
const style = view(
  Inputs.radio(["bar-chart", "grid", "spheres"], {
    label: "Heatmap style",
    value: "bar-chart",
  })
);
```

```js
const gridFill = view(
  Inputs.range([0.5, 1.5], { label: "Fill factor", step: 0.1 })
);
```

```js
const elevation = view(
  Inputs.range([0, 3], {
    label: "Elevation (grid & sphere)",
    step: 0.25,
    value: 2.75,
  })
);
```

<div id="smplr-container" style='height:500px; background-color: #F3F6F8;' />

```js
import { G } from "@mobily/ts-belt";
import { loadSmplrJs } from "@smplrspace/smplr-loader";

const smplr = await loadSmplrJs("esm", "dev");

const mappedco2Data = co2Data
  .map((d) => {
    const sensor = sensors.find((s) => s.id === d.uuid);
    if (!sensor) {
      return null;
    }
    return { ...d, position: sensor.position };
  })
  .filter((d) => G.isNotNullable(d));

const space = new smplr.Space({
  spaceId: "775d15d1-f26c-4e02-aa60-4a3a2a1bd785",
  clientToken: "pub_8c31c16e36804afe832b392f6e0d7843",
  containerId: "smplr-container",
});

const viewerReady = await space.startViewer({
  preview: false,
  allowModeChange: true,
  renderOptions: {
    backgroundColor: "#F3F6F8",
  },
  onError: (error) => console.error("Could not start viewer", error),
});
```

```js
space.addDataLayer({
  id: "hm",
  type: "heatmap",
  style,
  data: mappedco2Data,
  value: (d) => d.value,
  color: smplr.Color.numericScale({
    name: smplr.Color.NumericScale.RdYlGn,
    domain: [400, 800],
    invert: true,
  }),
  height: (v) => (v - 400) / 150,
  confidenceRadius: 9,
  gridSize: 0.6,
  gridFill,
  elevation,
});
```
