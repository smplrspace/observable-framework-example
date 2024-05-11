---
toc: false
theme: wide
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

## ${new Date(timeSlider).toUTCString()} <!-- TODO update to match TZ -->
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
    backgroundColor: 'gainsboro',
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

<div class="grid grid-cols-3 grid-rows-5" style="row-height: 150px">
<div class="card grid-colspan-1 grid-rowspan-4">
<h2>Carbon dioxide sensor readings (ppm)</h2>
${resize((width, height) => testHorizon(width, height))}
</div>
<div class="grid-colspan-2 grid-rowspan-3 card">
 <h2>Interior CO<sub>2</sub> concentrations</h2>

 ${styleInput}
 ${gridFillInput}
 ${elevationInput}
 <div id="smplr-container" style="height: 500px">
 ${smplrspaceHeatmap}
 </div>
</div>
<div class="card grid-rowspan-1 grid-colspan-1">
${drawBAN("test", "something")}
</div>
<div class="card grid-rowspan-1 grid-colspan-1"></div>
</div>

<div class="grid grid-cols-3">
<div class="card grid-colspan-2">
<h2>Full distribution of CO<sub>2</sub> readings</h2>
<h3>Includes all recorded values</h3>
${resize((width, height) => co2Histogram(width, height))}
</div>
<div class="card grid-colspan-1" style="padding: 0">
 ${Inputs.table(sensorDataClean, {columns: ["dateNew", "uuid", "value"], header: {dateNew: "Time", uuid: "Sensor ID", value: "CO2 concentration (ppm)"}})}
 </div>
 </div>

```js
const sensorDataClean = co2Data.map((d) => ({
  ...d,
  dateNew: d3.utcParse("%Y-%m-%dT%H:%M:%SZ")(d.ts),
}));
```

```js
// Filter sensor data to only match slider time
const timeSliderDateTime = (d3.utcParse("%a, %d %b %Y %H:%M:%S GMT")(new Date(timeSlider).toUTCString()));

const sensorDataFiltered = sensorDataClean.filter(d => new Date(d.dateNew).toUTCString() == new Date(timeSliderDateTime).toUTCString());
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

function testHorizon(width, height) {
  return Plot.plot({
  height: height - 20,
  width,
  margin: 0,
  marginTop: 30,
  x: {axis: "top", label: null},
  y: {domain: [0, step], axis: null},
  color: {scheme: "Greys"},
  marks: [
    d3.range(bands).map((band) => Plot.areaY(limitedSensorDataClean, {x: "dateNew", y: (d) => d.value - band * step, fy: "uuid", fill: band, opacity: 0.5, sort: "dateNew", clip: true})),
    Plot.ruleX([d3.utcParse("%a, %d %b %Y %H:%M:%S GMT")(new Date(timeSlider).toUTCString())]),
    Plot.axisFy({frameAnchor: "left", dx: -5, dy: 20, label: null, fontSize: 12, fontWeight: 400})
  ]
});
}
```

```js
function drawBAN(largeText, subText) {
  return Plot.plot({
  width,
  height: 600,
  marks: [
    Plot.text([largeText], {frameAnchor: "middle", fontSize: 30, lineWidth: 50, fontWeight: 600, dy: -30}),
    Plot.text([subText], {frameAnchor: "middle", fontSize: 18, dy: 20})
  ]
});
}
```

```js
function co2Histogram(width, height) {
  return Plot.plot({
    y: {type: "sqrt", grid: true, domain: [0, 3000]},
    x: {label: "CO2 concentration (ppm)", domain: [0, 1300]},
    color: {scheme: "RdYlGn", reverse: true, domain: d3.extent(sensorDataClean, d => d.value)},
    width,
    height: height - 40,
    marks: [
      Plot.rect([1], {x1: 400, x2: 1000, y1: 0, y2: 3000, opacity: 0.1}),
      Plot.rectY(sensorDataClean, Plot.binX({y: "count"}, {x: "value", opacity: 0.6})),
      Plot.text([`Normal indoor CO2 concentration range\n400 to 1,000 ppm`], {x: 1000, y: 3000, dy: 15, dx: -10, textAnchor: "end", fontStyle: "italic"})
    ]
  });
}
```