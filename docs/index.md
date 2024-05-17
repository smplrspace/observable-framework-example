---
toc: false
theme: wide
---

# Interior carbon dioxide readings

## Smplrspace and Observable Framework

```js
import { sensors } from "./data/sensors.js";
import { co2Data } from "./data/co2.js";
import { pipe, A, D } from "@mobily/ts-belt";
import { G } from "@mobily/ts-belt";
import { loadSmplrJs } from "@smplrspace/smplr-loader";

const smplr = await loadSmplrJs("esm", "dev");
```

```js
const mappedCo2Data = sensorDataFiltered
  .map((d) => {
    const sensor = sensors.find((s) => s.id === d.uuid);
    if (!sensor) {
      return null;
    }
    return { ...d, position: sensor.position };
  })
  .filter((d) => G.isNotNullable(d));
```

```js
// Inputs (for choosing smplrspace heatmap styling)
const styleInput = Inputs.radio(["bar-chart", "grid", "spheres"], {
  label: "Heatmap style",
  value: "bar-chart",
});

const style = Generators.input(styleInput);

const gridFillInput = Inputs.range([0.5, 1.5], {
  label: "Fill factor",
  step: 0.1,
});
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
    backgroundColor: "gainsboro",
  },
  onError: (error) => console.error("Could not start viewer", error),
});
```

```js
const smplrspaceHeatmap = space.addDataLayer({
  id: "hm",
  type: "heatmap",
  style,
  data: mappedCo2Data,
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

<div class="grid grid-cols-3 grid-rows-4" style="grid-auto-rows: auto;">
  <div class="card grid-colspan-1 grid-rowspan-4">
    <h2>Carbon dioxide sensor readings (ppm)</h2>
    ${resize((width, height) => testHorizon(width, height))}
  </div>
  <div class="card grid-rowspan-1 grid-colspan-1">
    <h2>Peak concentration: </h2>
    <span class="big">${d3.format(",")(maxValue.value) + " ppm"}</span>
    <br><br>
    <h3>${"Sensor: " + sensorsMaxValue[0] + " at " + new Date(maxValue.dateNew).toUTCString()}</h3>
  </div>
  <div class="card grid-rowspan-1 grid-colspan-1">
    <h2>Minimum concentration: </h2>
    <span class="big">${"Minimum concentration:", minValue.value + " ppm"}</span>
    <br><br>
    <h3>${"Recorded " + d3.format(",")(sensorsMinValue.length) + " total times by " + uniqueSensorsMinValue + " different sensors"}</h3>
  </div>
  <div class="grid-colspan-2 grid-rowspan-3 card">
    ${styleInput}
    ${gridFillInput}
    ${elevationInput}
    <div id="smplr-container" style="height: 450px">
      ${smplrspaceHeatmap}
    </div>
  </div>
</div>

<div class="grid grid-cols-2"">
  <div class="card grid-colspan-1" style="height: 300px">
    <h2>Distribution of CO<sub>2</sub> readings for this space</h2>
    ${resize((width, height) => co2Histogram(width, height))}
  </div>
  <div class="card grid-colspan-1">
    ${search}
    ${Inputs.table(searchValue, {columns: ["dateNew", "uuid", "value"], header: {dateNew: "Time", uuid: "Sensor ID", value: "CO2 concentration (ppm)"}})}
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
const timeSliderDateTime = d3.utcParse("%a, %d %b %Y %H:%M:%S GMT")(
  new Date(timeSlider).toUTCString()
);

const sensorDataFiltered = sensorDataClean.filter(
  (d) =>
    new Date(d.dateNew).toUTCString() ==
    new Date(timeSliderDateTime).toUTCString()
);

// Get object with maximum recorded value
const maxValue = sensorDataClean.reduce((max, val) =>
  max.value > val.value ? max : val
);

const sensorsMaxValue = sensorDataClean
  .filter((d) => d.value == maxValue.value)
  .map((d) => d.uuid);

const uniqueSensorsMaxValue = new Set(sensorsMaxValue).size;

// Get object with minimum recorded value
const minValue = sensorDataClean.reduce((min, val) =>
  min.value < val.value ? min : val
);

const sensorsMinValue = sensorDataClean
  .filter((d) => d.value == minValue.value)
  .map((d) => d.uuid);

const uniqueSensorsMinValue = new Set(sensorsMinValue).size;
```

```js
const timeSliderInput = Inputs.range(
  d3.extent(sensorDataClean.map((d) => d.dateNew)),
  { step: 600000 }
);

const timeSlider = Generators.input(timeSliderInput);

timeSliderInput.querySelector("input[type=number]").remove();
```

```js
const bands = 7;

console.log(d3.max(sensorDataClean, (d) => d.value));

const step = +(d3.max(sensorDataClean, (d) => d.value) / bands).toPrecision(2);

function testHorizon(width, height) {
  return Plot.plot({
    height: height,
    width,
    marginLeft: 100,
    marginRight: 0,
    marginTop: 30,
    x: { axis: "top", label: null },
    y: { domain: [0, step], axis: null },
    color: { scheme: "Greys" },
    marks: [
      d3.range(bands).map((band) =>
        Plot.areaY(sensorDataClean, {
          x: "dateNew",
          y: (d) => d.value - band * step,
          fy: "uuid",
          fill: band,
          opacity: 0.85,
          sort: "dateNew",
          clip: true,
        })
      ),
      Plot.ruleX(
        [
          d3.utcParse("%a, %d %b %Y %H:%M:%S GMT")(
            new Date(timeSlider).toUTCString()
          ),
        ],
        { stroke: "black" }
      ),
      Plot.axisFy({
        frameAnchor: "left",
        dx: -100,
        dy: 0,
        label: null,
        fontSize: 12,
        fontWeight: 400,
      }),
    ],
  });
}
```

```js
const breakpoints = [600, 1000];
const screenSize =
  window.innerWidth < breakpoints[0]
    ? 0
    : window.innerWidth < breakpoints[1]
    ? 1
    : 2;
```

```js
function co2Histogram(width, height) {
  return Plot.plot({
    y: { type: "sqrt", grid: true, domain: [0, 3000] },
    x: { label: "CO2 concentration (ppm)" },
    width,
    height: height - 20,
    marginBottom: 40,
    marginTop: 40,
    marks: [
      Plot.rectY(
        sensorDataClean,
        Plot.binX(
          { y: "count" },
          {
            x: "value",
            opacity: 0.95,
            fill: (d) => {
              // same value for whole bar width (20)
              const normalizedValue = Math.floor(d.value / 20) * 20;
              return co2ColorScale(normalizedValue);
            },
          }
        )
      ),
    ],
  });
}
```

```js
const co2ColorScale = (value) =>
  smplr.Color.cssToSmplrColor(
    smplr.Color.numericScale({
      name: smplr.Color.NumericScale.RdYlGn,
      domain: [400, 800],
      invert: true,
    })(value)
  );
```

```js
// search Input
const search = Inputs.search(sensorDataClean, {
  placeholder: "Search sensor data…",
});

const searchValue = Generators.input(search);
```
