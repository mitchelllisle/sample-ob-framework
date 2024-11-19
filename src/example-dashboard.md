---
theme: dashboard
title: Example dashboard
toc: false
---

# Rocket launches 🚀

<!-- Voronoi Char -->
```js
const data = [
  { numFiles: 23, numOutputRows: 150000, numOutputBytes: 1234567, cluster: 'normal' },
  { numFiles: 25, numOutputRows: 148500, numOutputBytes: 1245678, cluster: 'normal' },
  { numFiles: 22, numOutputRows: 151200, numOutputBytes: 1223456, cluster: 'normal' },
  { numFiles: 24, numOutputRows: 149800, numOutputBytes: 1256789, cluster: 'normal' },
  { numFiles: 26, numOutputRows: 152300, numOutputBytes: 1278901, cluster: 'upper-anomalous' },
  { numFiles: 21, numOutputRows: 147900, numOutputBytes: 1212345, cluster: 'lower-anomalous' },
  { numFiles: 24, numOutputRows: 149500, numOutputBytes: 1245678, cluster: 'normal' },
  { numFiles: 25, numOutputRows: 151800, numOutputBytes: 1267890, cluster: 'upper-anomalous' },
  { numFiles: 23, numOutputRows: 148700, numOutputBytes: 1234567, cluster: 'normal' },
  { numFiles: 22, numOutputRows: 150200, numOutputBytes: 1223456, cluster: 'normal' },
  { numFiles: 24, numOutputRows: 149300, numOutputBytes: 1245678, cluster: 'normal' },
  { numFiles: 23, numOutputRows: 151500, numOutputBytes: 1234567, cluster: 'normal' },
  { numFiles: 25, numOutputRows: 152000, numOutputBytes: 1267890, cluster: 'upper-anomalous' }
];
```

```js
function voronoicChart(data, {width} = {}) {
  return Plot.plot({
      title: "Anomalies",
      width,
      height: 500,
      marginTop: 30,
      marginBottom: 40,
      marginLeft: 60,
      color: {legend: true},
      marks: [
        Plot.voronoi(data, {x: "numOutputRows", y: "numOutputBytes", fill: "cluster", fillOpacity: 0.2, stroke: "white"}),
        Plot.dot(data, {x: "numOutputRows", y: "numOutputBytes", fill: "cluster", r: 5})
      ]
  })
};
```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => voronoicChart(data, {width}))}
  </div>
</div>

<!-- Load and transform the data -->

```js
const launches = FileAttachment("data/launches.csv").csv({typed: true});
```

<!-- A shared color scale for consistency, sorted by the number of launches -->

```js
const color = Plot.scale({
  color: {
    type: "categorical",
    domain: d3.groupSort(launches, (D) => -D.length, (d) => d.state).filter((d) => d !== "Other"),
    unknown: "var(--theme-foreground-muted)"
  }
});
```

<!-- Cards with big numbers -->

<div class="grid grid-cols-4">
  <div class="card">
    <h2>United States 🇺🇸</h2>
    <span class="big">${launches.filter((d) => d.stateId === "US").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Russia 🇷🇺 <span class="muted">/ Soviet Union</span></h2>
    <span class="big">${launches.filter((d) => d.stateId === "SU" || d.stateId === "RU").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>China 🇨🇳</h2>
    <span class="big">${launches.filter((d) => d.stateId === "CN").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Other</h2>
    <span class="big">${launches.filter((d) => d.stateId !== "US" && d.stateId !== "SU" && d.stateId !== "RU" && d.stateId !== "CN").length.toLocaleString("en-US")}</span>
  </div>
</div>

<!-- Plot of launch history -->

```js
function launchTimeline(data, {width} = {}) {
  return Plot.plot({
    title: "Launches over the years",
    width,
    height: 300,
    y: {grid: true, label: "Launches"},
    color: {...color, legend: true},
    marks: [
      Plot.rectY(data, Plot.binX({y: "count"}, {x: "date", fill: "state", interval: "year", tip: true})),
      Plot.ruleY([0])
    ]
  });
}
```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => launchTimeline(launches, {width}))}
  </div>
</div>

<!-- Plot of launch vehicles -->

```js
function vehicleChart(data, {width}) {
  return Plot.plot({
    title: "Popular launch vehicles",
    width,
    height: 300,
    marginTop: 0,
    marginLeft: 50,
    x: {grid: true, label: "Launches"},
    y: {label: null},
    color: {...color, legend: true},
    marks: [
      Plot.rectX(data, Plot.groupY({x: "count"}, {y: "family", fill: "state", tip: true, sort: {y: "-x"}})),
      Plot.ruleX([0])
    ]
  });
}
```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => vehicleChart(launches, {width}))}
  </div>
</div>

Data: Jonathan C. McDowell, [General Catalog of Artificial Space Objects](https://planet4589.org/space/gcat)
