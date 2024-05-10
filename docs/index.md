---
toc: false
---

# Smplrspace on Observable
## Carbon dioxide sensor data

```js
import { sensors } from "./data/sensors.js";
import { co2Data } from "./data/co2.js";
import { pipe, A, D } from "@mobily/ts-belt";
import { G } from "@mobily/ts-belt";
import { loadSmplrJs } from "@smplrspace/smplr-loader";

const smplr = await loadSmplrJs("esm", "dev");
```

```js
const mappedco2Data = sensorDataFiltered
  .map((d) => {
    const sensor = sensors.find((s) => s.id === d.uuid);
    if (!sensor) {
      return null;
    }
    return { ...d, position: sensor.position};
  })
  .filter((d) => G.isNotNullable(d))
;
```

```js
// Inputs (for choosing smplrspace heatmap styling)
const styleInput = Inputs.radio(["bar-chart", "grid", "spheres"], {
    label: "Heatmap style",
    value: "bar-chart",
  });

const style = Generators.input(styleInput);

const gridFillInput = Inputs.range([0.5, 1.5], { label: "Fill factor", step: 0.1 })
;

const gridFill = Generators.input(gridFillInput);

const elevationInput = Inputs.range([0, 3], {
    label: "Elevation (grid & sphere)",
    step: 0.25,
    value: 2.75,
  });

const elevation = Generators.input(elevationInput);
```

<h2>${new Date(timeSlider).toUTCString()}</h2> <!-- TODO update to match TZ -->
${timeSliderInput}

```js
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
const smplrspaceHeatmap = space.addDataLayer({
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

<div class="grid grid-cols-3">
<div class="card grid-colspan-1">
<h2>Carbon dioxide sensor readings (ppm)</h2>
${resize(width => testHorizon(width))}
</div>
<div class="grid-colspan-2 card">
 <h2>Interior CO<sub>2</sub> concentrations</h2>
 ${styleInput}
 ${gridFillInput}
 ${elevationInput}
 <div id="smplr-container">
 ${smplrspaceHeatmap}
 </div>
</div>
<div>
 ${Inputs.table(sensorDataFiltered)}
 </div>
</div>

```js
const sensorDataClean = co2Data.map((d) => ({
  ...d,
  dateNew: d3.utcParse("%Y-%m-%dT%H:%M:%SZ")(d.ts),
}));
```

```js echo
// Filter sensor data to only match slider time
const timeSliderDateTime = (d3.utcParse("%a, %d %b %Y %H:%M:%S GMT")(new Date(timeSlider).toUTCString()));

const sensorDataFiltered = sensorDataClean.filter(d => new Date(d.dateNew).toUTCString() == new Date(timeSliderDateTime).toUTCString());

view(Inputs.table(sensorDataFiltered));
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
const timeSliderInput = Inputs.range(d3.extent(limitedSensorDataClean.map(d => d.dateNew)), {step: 600000});

const timeSlider = Generators.input(timeSliderInput);

timeSliderInput.querySelector("input[type=number]").remove();
```

```js
const bands = 5;

const step = +(d3.max(limitedSensorDataClean, (d) => d.value) / bands).toPrecision(2);

function testHorizon(width) {
  return Plot.plot({
  height: 800,
  width,
  margin: 10,
  marginTop: 30,
  x: {axis: "top", label: null},
  y: {domain: [0, step], axis: null},
  color: {scheme: "greys" },
  marks: [
    d3.range(bands).map((band) => Plot.areaY(limitedSensorDataClean, {x: "dateNew", y: (d) => d.value - band * step, fy: "uuid", fill: band, opacity: 0.2, sort: "dateNew", clip: true})),
    Plot.ruleX([d3.utcParse("%a, %d %b %Y %H:%M:%S GMT")(new Date(timeSlider).toUTCString())]),
    Plot.axisFy({frameAnchor: "left", dx: -5, dy: 20, fill: "currentColor", textStroke: "white", label: null, fontSize: 15})
  ]
});
}
```

<div class="grid grid-cols-3">
</div>