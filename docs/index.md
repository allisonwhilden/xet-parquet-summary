---
toc: false
theme: [light, dark, wide]
sql: 
    weather: weather.parquet
---
# Parquet summary

<style>
body {
    font-family: -apple-system, "system-ui", "avenir next", avenir, helvetica, "helvetica neue", ubuntu, roboto, noto, "segoe ui", arial, sans-serif;
    font-size: 14px
}
.columns {
    display: flex;
    flex-direction: row;
    justify-content: start;
    gap: 20px;
    padding-top: 40px;
    overflow: scroll;
}

h4 {
    font-size: 13px;
    line-height: 1.2;
}

h5 {
    font-size: 11px;
    line-height: 1;
}

.column {
    display: flex;
    flex-direction: column;
    justify-content: start;
    min-width: 200px;
}

.stat {
    border-top: 1px solid #DFE0E0;
    padding: 8px 0;
}
</style>

```js
import { vgplot } from "./components/mosaic.js";
const weather = await FileAttachment("weather.parquet").url();
const vg = vgplot(vg => [ vg.loadParquet("weather", weather) ]);
const db = DuckDBClient.of({weather: FileAttachment("weather.parquet")});
const weatherParquet = FileAttachment("weather.parquet").parquet();

```

```js
// a selection instance to manage selected intervals from each plot
const $brush = vg.Selection.crossfilter();
```

```js
   var column_array = db.sql`DESCRIBE weather`
```

```js
    html`<div class="columns">${column_array.map((column) => ColumnDisplay(column))}</div>`
```



```js
function ColumnDisplay(column) {
    const continuousTypes = ["DOUBLE"]
    if (continuousTypes.includes(column.column_type)) {
        
        return (
            html`<div class="column">
                    <h4>${column.column_name}</h4>
                    <div>
                        ${  vg.plot(
                            vg.rectY(
                            vg.from("weather", { filterBy: $brush }),
                            { x: vg.bin(column.column_name), y: vg.count(), fill: "steelblue", inset: 0.5 }
                            ),
                            vg.intervalX({ as: $brush }),
                            vg.xDomain(vg.Fixed),
                            vg.yAxis(null),
                            vg.xLabel(null),
                            vg.width(200),
                            vg.height(150),
                        )}
                    </div>
                    <div class="stat">
                        <h5>Average</h5>
                        ${d3.mean(weatherParquet, d => d[column.column_name])}
                    </div>
                    <div class="stat">
                        <h5>Standard Deviation</h5>
                        ${d3.deviation(weatherParquet, d => d[column.column_name])}
                    </div>
                    <div class="stat">
                        <h5>Variance</h5>
                        ${d3.variance(weatherParquet, d => d[column.column_name])}
                    </div>
                </div>`
        )
    } else 
        return (
            html`<div class="column">
                <h4>${column.column_name}</h4>
                <div>
                    Top 10 Values
                    ${vg.plot(
                        vg.barX(
                            vg.from("weather", {filterBy: $brush}),
                            {
                                x: vg.count(),
                                y: column.column_name,
                                fill: "steelblue",
                                sort: {y: "-x", limit: 10},
                            }),  
                        vg.text(
                            vg.from("weather", {filterBy: $brush}),
                            {x: vg.count(), y: column.column_name, text: vg.count(), fill: "var(--theme-foreground)", dx: 10}),
                        vg.yLabel(null),
                        vg.xAxis(null),
                        vg.width(200),
                        vg.height(300),
                    )}
                </div>
            </div>`
        )
}
```


View source on [GitHub](https://github.com/allisonwhilden/xet-parquet-summary)