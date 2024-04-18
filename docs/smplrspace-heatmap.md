# Smplrspace heatmap

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

<div id="smplr-container" style='height:500px; background-color: pink;' />

```js
import { G } from "@mobily/ts-belt";
import { loadSmplrJs } from "@smplrspace/smplr-loader";

const smplr = await loadSmplrJs("esm", "dev");

const mappedSensorsData = sensorsData
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
  data: mappedSensorsData,
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

<!-- DATA -->

```js
const sensors = [
  {
    id: "de9844aef33d",
    name: "1",
    position: {
      x: 120.72363861531221,
      z: -55.37731272654325,
      elevation: 0.009999999776482582,
      levelIndex: 0,
    },
  },
  {
    id: "cde60d0f9c4b",
    name: "2",
    position: {
      x: 120.31616658822531,
      z: -40.32848595282147,
      elevation: 0.009999999776486135,
      levelIndex: 0,
    },
  },
  {
    id: "e22831cdda28",
    name: "3",
    position: {
      x: 117.51274810528558,
      z: -38.38687380878737,
      elevation: 0.009999999776482582,
      levelIndex: 0,
    },
  },
  {
    id: "f7262ecc48b9",
    name: "4",
    position: {
      x: 117.82239335232062,
      z: -49.51764202937709,
      elevation: 0.009999999776486135,
      levelIndex: 0,
    },
  },
  {
    id: "fdee6f5c3609",
    name: "5",
    position: {
      x: 136.54854211915313,
      z: -56.12197980280458,
      elevation: 0.009999999776482582,
      levelIndex: 0,
    },
  },
  {
    id: "fa4ced683b3e",
    name: "6",
    position: {
      x: 97.77430667927675,
      z: -57.3023438052912,
      elevation: 0.009999999776482582,
      levelIndex: 0,
    },
  },
  {
    id: "fb1bd4c58976",
    name: "7",
    position: {
      x: 107.511986825776,
      z: -53.488347539106115,
      elevation: 0.009999999776482582,
      levelIndex: 0,
    },
  },
  {
    id: "c3b82bf22edc",
    name: "8",
    position: {
      x: 116.18496766013054,
      z: -49.64956961177282,
      elevation: 0.009999999776496793,
      levelIndex: 0,
    },
  },
  {
    id: "cae274222f6c",
    name: "9",
    position: {
      x: 126.53886269246092,
      z: -60.5842980863642,
      elevation: 0.009999999776503898,
      levelIndex: 0,
    },
  },
  {
    id: "f34a076b36a3",
    name: "10",
    position: {
      x: 117.57374279582795,
      z: -48.555304314518494,
      elevation: 0.009999999776489688,
      levelIndex: 0,
    },
  },
  {
    id: "c94f3f0243a7",
    name: "11",
    position: {
      x: 115.84157357866134,
      z: -48.53316088837151,
      elevation: 0.009999999776503898,
      levelIndex: 0,
    },
  },
  {
    id: "f3524c41d309",
    name: "12",
    position: {
      x: 112.93403943261109,
      z: -46.77269913954395,
      elevation: 0.009999999776489688,
      levelIndex: 0,
    },
  },
  {
    id: "d7942311a750",
    name: "13",
    position: {
      x: 120.64810233459582,
      z: -46.60908426577184,
      elevation: 0.009999999776482582,
      levelIndex: 0,
    },
  },
  {
    id: "e5fb7587a358",
    name: "14",
    position: {
      x: 116.20911006309461,
      z: -38.400813282810724,
      elevation: 0.009999999776496793,
      levelIndex: 0,
    },
  },
  {
    id: "c5a4f171c1f8",
    name: "15",
    position: {
      x: 107.55121161309727,
      z: -37.61779228809321,
      elevation: 0.009999999776511004,
      levelIndex: 0,
    },
  },
  {
    id: "cc51c86dfc78",
    name: "16",
    position: {
      x: 128.28534381725393,
      z: -42.1580788002604,
      elevation: 0.009999999776489688,
      levelIndex: 0,
    },
  },
];
```

```js
const sensorsData = [
  {
    ts: "2024-01-16T11:10:00Z",
    value: 628,
    uuid: "de9844aef33d",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 495,
    uuid: "cde60d0f9c4b",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 400,
    uuid: "e22831cdda28",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 400,
    uuid: "f7262ecc48b9",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 400,
    uuid: "fdee6f5c3609",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 552,
    uuid: "fa4ced683b3e",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 1276,
    uuid: "fb1bd4c58976",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 486,
    uuid: "c3b82bf22edc",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 449,
    uuid: "cae274222f6c",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 616,
    uuid: "f34a076b36a3",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 550,
    uuid: "c94f3f0243a7",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 400,
    uuid: "f3524c41d309",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 400,
    uuid: "d7942311a750",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 652,
    uuid: "e5fb7587a358",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 820,
    uuid: "c5a4f171c1f8",
  },
  {
    ts: "2024-01-16T11:10:00Z",
    value: 548,
    uuid: "cc51c86dfc78",
  },
];
```
