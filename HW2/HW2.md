# Homework 2: Choosing Encodings

**Variables: Miles per Gallon, Weight, Year**

- What is the data type for each variable?
    - Miles per Gallon: Continuous Numerical
    - Weight: Continuous Numerical
    - Year: Discreate Numerical
- Could this variable be treated as a different data type?
    - Miles per Gallon: Ordinal Categorical, by putting them to buckets/brackets
    - Weight: Ordinal Categorical, by putting them to buckets/brackets
    - Year: Ordinal Categorical, as individual year or decade
- In what contexts might it be helpful to consider the variable as one type or another and why?
    - As numerical when using position as encoding, since it support arbitrary values.
    - As categorical when using encodings like color and shape, since they're limited and too much variations could decrease effectiveness.
- Which encoding channels are appropriate (expressive) for this variable?
  Position is the most appropriate encoding channel for this variable.
- Which encoding channels are best for prioritizing (effective)/deprioritizing each variable?
    - Size
    - Color Saturation


```python
from datetime import datetime

import pandas as pd
import altair as alt
from vega_datasets import data as vega_data

source = vega_data.cars()
source.shape
```


```python
source
```


```python
cars_in_80s = source.loc[source["Year"] >= datetime.fromisoformat("1980-01-01")]
cars_in_80s
```


```python
# MPG and Weight
mpg_weight_chart = alt.Chart(cars_in_80s).mark_point().encode(
    y=alt.Y("Miles_per_Gallon:Q", title="Miles per Gallon"),
    x=alt.X("Weight_in_lbs:Q", title="Weight (lbs)"),
    color="Year:O"
)

mpg_weight_chart.properties(width=600, height=600, title="MPG vs Weight of Cars in 1980s")
```

- What are the three encoding channels you chose for this graphic?
    - X-position for Weight
    - Y-position for Miles per Gallon
    - Color for Year

- What are the marks you chose for this graphic? Why are these marks applicable and useful here? What others would have been appropriate?

  I used Point as marks so the result product is a scatter plot. Since there are multiple entries in the data and each of them are independent.

- Did you make any changes to the data type, scales, or other properties of the graph? How do these changes aid in interpretation?

  I chose to selectively display data point by filter by years, as the entire dataset is too large, result in cluttered graph. Since Miles per Gallon and Weight is largely independent of year, so the message is same as showing the entire population.

- Why does this graphic prioritize the relationship you wanted to showcase?

  Usually, position encoding is the most effective, and I use it for the two variables of focus. And the Year as color is secondary to position.

- What is a one sentence statement that your graphic supports?

  There is a negative correlation between Miles per Gallon and Weight, for which when weight increases the Miles per Gallon generally decreases.

- How does what we learned in class inform the way you made your graphic?

  The comparison between encoding channels basically give me the answer on which encoding channel to use for, for which data visualization is about maximizing effectiveness and expressiveness rather than be creative. And rankings among encoding channels help me to choose the best one for each variable for prioritization.


```python
# Weight and Years
weight_mpg_combined_stats = source.groupby("Year").agg({
    "Weight_in_lbs": ["min", "max", "mean"],
    "Miles_per_Gallon": ["mean"]
}).reset_index()

weight_mpg_combined_stats.columns = ["Year", "Weight_min", "Weight_max", "Weight_mean", "MPG_mean"]

combined_chart_base = alt.Chart(weight_mpg_combined_stats).encode(x=alt.X("Year:T", title="Year"))

weight_scale = alt.Scale(domain=[weight_mpg_combined_stats["Weight_min"].min(), weight_mpg_combined_stats["Weight_max"].max()])

combined_weight_mpg_chart = alt.layer(
    combined_chart_base.mark_area(opacity=0.3).encode(
        y=alt.Y("Weight_min:Q", scale=weight_scale, axis=None),
        y2=alt.Y2("Weight_max:Q"),
        color=alt.datum("Weight Min-Max Range")
    ),
    combined_chart_base.mark_line().encode(
        y=alt.Y("Weight_mean:Q", scale=weight_scale, title="Weight (lbs)", axis=alt.Axis(orient="left")),
        color=alt.datum("Mean Weight"),
        strokeDash=alt.datum("Mean Weight")
    ),
    combined_chart_base.mark_line().encode(
        y=alt.Y("MPG_mean:Q", title="Miles per Gallon", axis=alt.Axis(orient="right")),
        color=alt.datum("Mean MPG"),
        strokeDash=alt.datum("Mean MPG")
    )
).resolve_scale(
    y="independent"
).configure_range(
    category=["lightblue", "blue", "pink"],
    strokeDash=[[1, 0], [1, 0], [5, 5]]
).properties(width=600, height=400, title="Weight and Miles per Gallon Over Years")

combined_weight_mpg_chart

```





<style>
  #altair-viz-0ef397c9c7884686b364ff64b734af2e.vega-embed {
    width: 100%;
    display: flex;
  }

  #altair-viz-0ef397c9c7884686b364ff64b734af2e.vega-embed details,
  #altair-viz-0ef397c9c7884686b364ff64b734af2e.vega-embed details summary {
    position: relative;
  }
</style>
<div id="altair-viz-0ef397c9c7884686b364ff64b734af2e"></div>
<script type="text/javascript">
  var VEGA_DEBUG = (typeof VEGA_DEBUG == "undefined") ? {} : VEGA_DEBUG;
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-0ef397c9c7884686b364ff64b734af2e") {
      outputDiv = document.getElementById("altair-viz-0ef397c9c7884686b364ff64b734af2e");
    }

    const paths = {
      "vega": "https://cdn.jsdelivr.net/npm/vega@6?noext",
      "vega-lib": "https://cdn.jsdelivr.net/npm/vega-lib?noext",
      "vega-lite": "https://cdn.jsdelivr.net/npm/vega-lite@6.1.0?noext",
      "vega-embed": "https://cdn.jsdelivr.net/npm/vega-embed@7?noext",
    };

    function maybeLoadScript(lib, version) {
      var key = `${lib.replace("-", "")}_version`;
      return (VEGA_DEBUG[key] == version) ?
        Promise.resolve(paths[lib]) :
        new Promise(function(resolve, reject) {
          var s = document.createElement('script');
          document.getElementsByTagName("head")[0].appendChild(s);
          s.async = true;
          s.onload = () => {
            VEGA_DEBUG[key] = version;
            return resolve(paths[lib]);
          };
          s.onerror = () => reject(`Error loading script: ${paths[lib]}`);
          s.src = paths[lib];
        });
    }

    function showError(err) {
      outputDiv.innerHTML = `<div class="error" style="color:red;">${err}</div>`;
      throw err;
    }

    function displayChart(vegaEmbed) {
      vegaEmbed(outputDiv, spec, embedOpt)
        .catch(err => showError(`Javascript Error: ${err.message}<br>This usually means there's a typo in your chart specification. See the javascript console for the full traceback.`));
    }

    if(typeof define === "function" && define.amd) {
      requirejs.config({paths});
      let deps = ["vega-embed"];
      require(deps, displayChart, err => showError(`Error loading script: ${err.message}`));
    } else {
      maybeLoadScript("vega", "6")
        .then(() => maybeLoadScript("vega-lite", "6.1.0"))
        .then(() => maybeLoadScript("vega-embed", "7"))
        .catch(showError)
        .then(() => displayChart(vegaEmbed));
    }
  })({"config": {"view": {"continuousWidth": 300, "continuousHeight": 300}, "range": {"category": ["lightblue", "blue", "pink"], "strokeDash": [[1, 0], [1, 0], [5, 5]]}}, "layer": [{"mark": {"type": "area", "opacity": 0.3}, "encoding": {"color": {"datum": "Weight Min-Max Range"}, "x": {"field": "Year", "title": "Year", "type": "temporal"}, "y": {"axis": null, "field": "Weight_min", "scale": {"domain": [1613.0, 5140.0]}, "type": "quantitative"}, "y2": {"field": "Weight_max"}}}, {"mark": {"type": "line"}, "encoding": {"color": {"datum": "Mean Weight"}, "strokeDash": {"datum": "Mean Weight"}, "x": {"field": "Year", "title": "Year", "type": "temporal"}, "y": {"axis": {"orient": "left"}, "field": "Weight_mean", "scale": {"domain": [1613.0, 5140.0]}, "title": "Weight (lbs)", "type": "quantitative"}}}, {"mark": {"type": "line"}, "encoding": {"color": {"datum": "Mean MPG"}, "strokeDash": {"datum": "Mean MPG"}, "x": {"field": "Year", "title": "Year", "type": "temporal"}, "y": {"axis": {"orient": "right"}, "field": "MPG_mean", "title": "Miles per Gallon", "type": "quantitative"}}}], "data": {"name": "data-e413e256781d33d3fa066f7993c6eb6e"}, "height": 400, "resolve": {"scale": {"y": "independent"}}, "title": "Weight and Miles per Gallon Over Years", "width": 600, "$schema": "https://vega.github.io/schema/vega-lite/v6.1.0.json", "datasets": {"data-e413e256781d33d3fa066f7993c6eb6e": [{"Year": "1970-01-01T00:00:00", "Weight_min": 1835, "Weight_max": 4732, "Weight_mean": 3441.3142857142857, "MPG_mean": 17.689655172413794}, {"Year": "1971-01-01T00:00:00", "Weight_min": 1613, "Weight_max": 5140, "Weight_mean": 2960.344827586207, "MPG_mean": 21.25}, {"Year": "1972-01-01T00:00:00", "Weight_min": 2100, "Weight_max": 4633, "Weight_mean": 3237.714285714286, "MPG_mean": 18.714285714285715}, {"Year": "1973-01-01T00:00:00", "Weight_min": 1867, "Weight_max": 4997, "Weight_mean": 3419.025, "MPG_mean": 17.1}, {"Year": "1974-01-01T00:00:00", "Weight_min": 1649, "Weight_max": 4699, "Weight_mean": 2877.925925925926, "MPG_mean": 22.703703703703702}, {"Year": "1975-01-01T00:00:00", "Weight_min": 1795, "Weight_max": 4668, "Weight_mean": 3176.8, "MPG_mean": 20.266666666666666}, {"Year": "1976-01-01T00:00:00", "Weight_min": 1795, "Weight_max": 4380, "Weight_mean": 3078.735294117647, "MPG_mean": 21.573529411764707}, {"Year": "1977-01-01T00:00:00", "Weight_min": 1825, "Weight_max": 4335, "Weight_mean": 2997.3571428571427, "MPG_mean": 23.375}, {"Year": "1978-01-01T00:00:00", "Weight_min": 1800, "Weight_max": 4080, "Weight_mean": 2861.8055555555557, "MPG_mean": 24.061111111111114}, {"Year": "1979-01-01T00:00:00", "Weight_min": 1915, "Weight_max": 4360, "Weight_mean": 3055.344827586207, "MPG_mean": 25.093103448275862}, {"Year": "1980-01-01T00:00:00", "Weight_min": 1835, "Weight_max": 3381, "Weight_mean": 2436.655172413793, "MPG_mean": 33.69655172413793}, {"Year": "1982-01-01T00:00:00", "Weight_min": 1755, "Weight_max": 3725, "Weight_mean": 2492.2131147540986, "MPG_mean": 31.045}]}}, {"mode": "vega-lite"});
</script>



- What are the three encoding channels you chose for this graphic?
    - X-position for Year
    - Y-position for Weight and Miles per Gallon
    - Line thickness style for distinguish between Weight and Miles per Gallon series; and color between Mean and Min-Max Range

- What are the marks you chose for this graphic? Why are these marks applicable and useful here? What others would have been appropriate?

  I used Link as marks to make this multi-series line chart. Since the input variable is continuous, so the line chart is good to show the trend.

- Did you make any changes to the data type, scales, or other properties of the graph? How do these changes aid in interpretation?

  Since the Year is discrete and there are multiple entries of the same year, I aggregate the data by year to show the overall trend. Without aggregation, the graph would be cluttered won't be expressive to show patterns.

- Why does this graphic prioritize the relationship you wanted to showcase?

  Usually, position encoding is the most effective, and I use it for the two variables of focus.
  As well as based on Left-Right reading pattern, I put the Weight on the left Y-axis and Miles per Gallon on the right Y-axis to prioritize Miles per Gallon.
  Also, the relationship between different series is categorical, so I used color saturation to weaken the non-primary series (i.e. Miles per Gallon) to de-prioritize it.

- What is a one sentence statement that your graphic supports?

  The average weight and maximum of cars has generally decreasing over the years.

- How does what we learned in class inform the way you made your graphic?

  I learned that the graphic doesn't need to show all the data points and can transform the data to better express the relationship, all for the purpose of effective communication and highlighting the statement to support.

Alternatively, I think the chart can be improved by excluding the Miles per Gallon variable, to make it more focused on Weight only, since the line chart requires all variables to be transformed to the same format and display in the same way.


```python
# Miles per Gallon and Year
mpg_weight_combined_stats = source.groupby("Year").agg({
    "Weight_in_lbs": ["mean"],
    "Miles_per_Gallon": ["min", "max", "mean"],
}).reset_index()

mpg_weight_combined_stats.columns = ["Year", "Weight_mean", "MPG_min", "MPG_max", "MPG_mean"]

mpg_weight_base = alt.Chart(mpg_weight_combined_stats).encode(x=alt.X("Year:T", title="Year"))

mpg_layers = alt.layer(
    mpg_weight_base.mark_area(opacity=0.3).encode(
        y=alt.Y("MPG_min:Q", title="Miles per Gallon"),
        y2="MPG_max:Q",
        color=alt.datum("Miles per Gallon Range")
    ),
    mpg_weight_base.mark_line().encode(
        y=alt.Y("MPG_mean:Q"),
        color=alt.datum("Mean MPG"),
        strokeDash=alt.datum("Mean MPG")
    )
)
weight_layer = mpg_weight_base.mark_line().encode(
    y=alt.Y("Weight_mean:Q", title="Weight (lbs)"),
    color=alt.datum("Mean Weight"),
    strokeDash=alt.datum("Mean Weight")
)

combined_mpg_weight_chart = alt.layer(
    mpg_layers,
    weight_layer
).resolve_scale(
    y="independent"
).configure_range(
    category=["lightblue", "blue", "pink"],
    strokeDash=[[1, 0], [1, 0], [5, 5]]
).properties(
    width=600,
    height=400,
    title="Miles per Gallon and Weight Over Time"
)

combined_mpg_weight_chart

```

- What are the three encoding channels you chose for this graphic?
    - X-position for Year
    - Y-position for Miles per Gallon and Weight
    - Line thickness style for distinguish between Weight and Miles per Gallon series; and color between Mean and Min-Max Range

- What are the marks you chose for this graphic? Why are these marks applicable and useful here? What others would have been appropriate?

  I used Link as marks to make this multi-series line chart. Since the input variable is continuous, so the line chart is good to show the trend.

- Did you make any changes to the data type, scales, or other properties of the graph? How do these changes aid in interpretation?

  Since the Year is discrete and there are multiple entries of the same year, I aggregate the data by year to show the overall trend. Without aggregation, the graph would be cluttered won't be expressive to show patterns.

- Why does this graphic prioritize the relationship you wanted to showcase?

  Usually, position encoding is the most effective, and I use it for the two variables of focus.
  As well as based on Left-Right reading pattern, I put the Miles per Gallon on the left Y-axis and Weight on the right Y-axis to prioritize Weight.
  Also, the relationship between different series is categorical, so I used color saturation to weaken the non-primary series (i.e. Weight) to de-prioritize it.

- What is a one sentence statement that your graphic supports?

  The average Miles per Gallon has generally increasing over the years, along with the lower-bound and upper-bound of Miles per Gallon shifting upward.

- How does what we learned in class inform the way you made your graphic?

  I learned that the graphic doesn't need to shown all the data points and can transform the data to better express the relationship, all for the purpose of effective communication and highlighting the statement to support.


Alternatively, I think the chart can be improved by excluding the Weight variable, to make it more focused on Miles per Gallon only, since the line chart requires all variables to be transformed to the same format and display in the same way.


```python

```


```python
# Ineffective -- Weight and Year
ineffective_weight_year_chart = alt.Chart(source).mark_point().encode(
    x=alt.X("Year:T", title="Year"),
    y=alt.Y("Weight_in_lbs:Q", title="Weight (lbs)"),
)
ineffective_weight_year_chart.properties(width=600, height=400, title="Weight of Cars Over Years")
```

- What are the three encoding channels you chose for this graphic?
    - X-position for Year
    - Y-position for Weight
    - None for Miles per Gallon

- What are the marks you chose for this graphic? Why are these marks applicable and useful here? What others would have been appropriate?

  I used Point as marks so the result product is a scatter plot. Since there are multiple entries in the data and each of them are independent.

- Did you make any changes to the data type, scales, or other properties of the graph? How do these changes aid in interpretation?

  I kept the data as-is without aggregation or changing in scale, to keep the chart expressive in terms of accurately showing the original data.

- Why does this graphic prioritize the relationship you wanted to showcase?

  Usually, position encoding is the most effective, and I use it for the two variables of focus.

- What is a one sentence statement that your graphic supports?

  The maximum of cars has generally decreasing over the years, and many cars have similar weight to another.

- How does what we learned in class inform the way you made your graphic?

  I learned the difference between expressive and effective, and how to balance them. In this case, the chart is expressive since it shows all data points, but not effective since the cluttered points make it hard to see patterns.
