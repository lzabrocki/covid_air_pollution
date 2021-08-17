---
title:  "Comorbodities & Long-Term Air Pollution Exposure"
description: |
  Detailled Script.
author:
  - name: Léo Zabrocki 
    url: https://www.parisschoolofeconomics.eu/
    affiliation: Paris School of Economics
    affiliation_url: https://www.parisschoolofeconomics.eu/
date: "2021-08-17"
output:
  distill::distill_article:
    keep_md: true
    toc: true
    toc_depth: 2
---

<style>
body {
text-align: justify}
</style>



# Required Packages and Loading Data

To reproduce exactly the `script_comorbidites_air_pollution.html` document, you first need to have installed:

* the [R](https://www.r-project.org/) programming language on your computer 
* [RStudio](https://rstudio.com/), an integrated development environment for R, which will allow you to knit the `script_comorbidites_air_pollution.Rmd` file and interact with the R code chunks
* the [R Markdown](https://rmarkdown.rstudio.com/) package
* and the [Distill](https://rstudio.github.io/distill/) package which provides the template of this document. 

Once everything is set up, we need to load the following packages:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load required packages</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://github.com/rstudio/rmarkdown'>rmarkdown</a></span><span class='op'>)</span> <span class='co'># for creating the R Markdown document</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://yihui.org/knitr/'>knitr</a></span><span class='op'>)</span> <span class='co'># for creating the R Markdown document</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://here.r-lib.org/'>here</a></span><span class='op'>)</span> <span class='co'># for files paths organization</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://tidyverse.tidyverse.org'>tidyverse</a></span><span class='op'>)</span> <span class='co'># for data manipulation and visualization</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://modelr.tidyverse.org'>modelr</a></span><span class='op'>)</span>     <span class='co'># provides easy pipeline modeling functions</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://broom.tidymodels.org/'>broom</a></span><span class='op'>)</span>      <span class='co'># helps to tidy up model outputs</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://lubridate.tidyverse.org'>lubridate</a></span><span class='op'>)</span> <span class='co'># for manipulating date variables</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='http://www.rforge.net/Cairo/'>Cairo</a></span><span class='op'>)</span> <span class='co'># for printing customed police of graphs</span>
</code></pre></div>

</div>


We also load our custom `ggplot2` theme for graphs:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load ggplot custom theme</span>
<span class='kw'><a href='https://rdrr.io/r/base/source.html'>source</a></span><span class='op'>(</span>
  <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
    <span class='st'>"2.scripts"</span>,
    <span class='st'>"4.custom_ggplot2_theme"</span>,
    <span class='st'>"script_custom_ggplot_theme.R"</span>
  <span class='op'>)</span>
<span class='op'>)</span>
</code></pre></div>

</div>


The theme is based on the fantastic [hrbrthemes](https://hrbrmstr.github.io/hrbrthemes/index.html) package. If you do not want to use this theme or are unable to install it because of fonts issues, you can use the `theme_mimimal()` included in the `ggplot2` package.

We load the data we use in our analysis:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load the data</span>
<span class='va'>data</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/base/readRDS.html'>readRDS</a></span><span class='op'>(</span><span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"3.data_for_analysis"</span>, <span class='st'>"data_analysis.rds"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


# Data Wrangling

We select the relevant variables:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>dplyr</span><span class='fu'>::</span><span class='fu'><a href='https://dplyr.tidyverse.org/reference/select.html'>select</a></span><span class='op'>(</span>
    <span class='va'>city</span>,
    <span class='va'>patient_id</span>,
    <span class='va'>dead</span><span class='op'>:</span><span class='va'>cancer</span>,
    <span class='va'>hbp</span>,
    <span class='va'>mean_2017_2019_no2</span>,
    <span class='va'>mean_2017_2019_o3</span>,
    <span class='va'>mean_2017_2019_pm10</span>,
    <span class='va'>mean_2017_2019_pm2p5</span>,
    <span class='va'>mean_2017_2019_so2</span>
  <span class='op'>)</span>
</code></pre></div>

</div>


We change the `sex` variable to numeric:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>sex <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='va'>sex</span> <span class='op'>==</span> <span class='st'>"male"</span>, <span class='fl'>1</span>, <span class='fl'>0</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


Number of observations by outcome:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># print the table</span>
<span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>dplyr</span><span class='fu'>::</span><span class='fu'><a href='https://dplyr.tidyverse.org/reference/select.html'>select</a></span><span class='op'>(</span><span class='va'>sex</span>, <span class='va'>hbp</span>, <span class='va'>diabetes</span>, <span class='va'>cardiac_disease_infarction</span>, <span class='va'>copd</span>, <span class='va'>renal_failure</span>, <span class='va'>cancer</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise_all</span><span class='op'>(</span><span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='op'>!</span><span class='fu'><a href='https://rdrr.io/r/base/NA.html'>is.na</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>cols <span class='op'>=</span> <span class='fu'>everything</span><span class='op'>(</span><span class='op'>)</span>, names_to <span class='op'>=</span> <span class='st'>"Comorbidity"</span>, values_to <span class='op'>=</span> <span class='st'>"Number of Non-Missing Observations"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>arrange</span><span class='op'>(</span><span class='op'>-</span><span class='va'>`Number of Non-Missing Observations`</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/pkg/knitr/man/kable.html'>kable</a></span><span class='op'>(</span>align <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"l"</span>, <span class='st'>"c"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>


|Comorbidity                | Number of Non-Missing Observations |
|:--------------------------|:----------------------------------:|
|renal_failure              |                659                 |
|cancer                     |                659                 |
|sex                        |                658                 |
|hbp                        |                658                 |
|diabetes                   |                658                 |
|copd                       |                658                 |
|cardiac_disease_infarction |                627                 |

</div>


We finally label variables:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='co'># rename patients' characteristics</span>
  <span class='fu'>rename</span><span class='op'>(</span>
    <span class='st'>"Sex"</span> <span class='op'>=</span> <span class='va'>sex</span>,
    <span class='st'>"HBP"</span> <span class='op'>=</span> <span class='va'>hbp</span>,
    <span class='st'>"Diabetes"</span> <span class='op'>=</span> <span class='va'>diabetes</span>,
    <span class='st'>"Cardiac Disease Infarction"</span> <span class='op'>=</span> <span class='va'>cardiac_disease_infarction</span>,
    <span class='st'>"COPD"</span> <span class='op'>=</span> <span class='va'>copd</span>,
    <span class='st'>"Renal Failure"</span> <span class='op'>=</span> <span class='va'>renal_failure</span>,
    <span class='st'>"Cancer"</span> <span class='op'>=</span> <span class='va'>cancer</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># rename air pollution variables</span>
  <span class='fu'>rename</span><span class='op'>(</span>
    <span class='st'>"NO2 Exposure (µg/m³)"</span> <span class='op'>=</span> <span class='va'>mean_2017_2019_no2</span>,
    <span class='st'>"O3 Exposure (µg/m³)"</span> <span class='op'>=</span> <span class='va'>mean_2017_2019_o3</span>,
    <span class='st'>"PM10 Exposure (µg/m³)"</span> <span class='op'>=</span> <span class='va'>mean_2017_2019_pm10</span>,
    <span class='st'>"PM2.5 Exposure (µg/m³)"</span> <span class='op'>=</span> <span class='va'>mean_2017_2019_pm2p5</span>,
    <span class='st'>"SO2 Exposure (µg/m³)"</span> <span class='op'>=</span> <span class='va'>mean_2017_2019_so2</span>
  <span class='op'>)</span> 
</code></pre></div>

</div>


# Logistic Regression Analysis

We first run the analysis for all comorbidites but cardiac disease infarction. We run a separate analysis for this comorbidity since no patients in Reims have it.

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>`NO2 Exposure (µg/m³)`</span><span class='op'>:</span><span class='va'>`SO2 Exposure (µg/m³)`</span><span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"pollutant"</span>,
    values_to <span class='op'>=</span> <span class='st'>"concentration"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>Sex</span>, <span class='va'>HBP</span><span class='op'>:</span><span class='va'>Cancer</span><span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"comorbidities"</span>,
    values_to <span class='op'>=</span> <span class='st'>"value_comorbidities"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>comorbidities</span> <span class='op'>!=</span> <span class='st'>"Cardiac Disease Infarction"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>comorbidities</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>nest</span><span class='op'>(</span><span class='op'>)</span>

<span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>&lt;-</span>
  <span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>logistic_model <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span>
    <span class='va'>data</span>,
    <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/stats/glm.html'>glm</a></span><span class='op'>(</span><span class='va'>value_comorbidities</span> <span class='op'>~</span> <span class='va'>concentration</span> <span class='op'>+</span> <span class='va'>city</span>, family <span class='op'>=</span> <span class='st'>"binomial"</span>,
             data <span class='op'>=</span> <span class='va'>.</span><span class='op'>)</span>
  <span class='op'>)</span><span class='op'>)</span>

<span class='co'># tidy regression ouputs</span>
<span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>&lt;-</span>
  <span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>logistic_model <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span><span class='va'>logistic_model</span>, <span class='op'>~</span> <span class='fu'>broom</span><span class='fu'>::</span><span class='fu'><a href='https://generics.r-lib.org/reference/tidy.html'>tidy</a></span><span class='op'>(</span><span class='va'>.</span>, conf.int <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># unnest results</span>
<span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>&lt;-</span>
  <span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>unnest</span><span class='op'>(</span><span class='va'>logistic_model</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>dplyr</span><span class='fu'>::</span><span class='fu'><a href='https://dplyr.tidyverse.org/reference/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>term</span> <span class='op'>==</span> <span class='st'>"concentration"</span><span class='op'>)</span>

<span class='co'># exponentiate</span>
<span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>&lt;-</span>
  <span class='va'>data_regression_analysis_wo_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='va'>estimate</span>, <span class='va'>conf.low</span>, <span class='va'>conf.high</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/Log.html'>exp</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


We then run the analysis for cardiac disease infarction but we drop patients from Reims:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_regression_analysis_cdi</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>`NO2 Exposure (µg/m³)`</span><span class='op'>:</span><span class='va'>`SO2 Exposure (µg/m³)`</span><span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"pollutant"</span>,
    values_to <span class='op'>=</span> <span class='st'>"concentration"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>Sex</span>, <span class='va'>HBP</span><span class='op'>:</span><span class='va'>Cancer</span><span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"comorbidities"</span>,
    values_to <span class='op'>=</span> <span class='st'>"value_comorbidities"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>comorbidities</span> <span class='op'>==</span> <span class='st'>"Cardiac Disease Infarction"</span> <span class='op'>&amp;</span>
           <span class='va'>city</span> <span class='op'>!=</span> <span class='st'>"Reims"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>comorbidities</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>nest</span><span class='op'>(</span><span class='op'>)</span>

<span class='va'>data_regression_analysis_cdi</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>logistic_model <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span>
    <span class='va'>data</span>,
    <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/stats/glm.html'>glm</a></span><span class='op'>(</span><span class='va'>value_comorbidities</span> <span class='op'>~</span> <span class='va'>concentration</span> <span class='op'>+</span> <span class='va'>city</span>, family <span class='op'>=</span> <span class='st'>"binomial"</span>,
             data <span class='op'>=</span> <span class='va'>.</span><span class='op'>)</span>
  <span class='op'>)</span><span class='op'>)</span>

<span class='co'># tidy regression ouputs</span>
<span class='va'>data_regression_analysis_cdi</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>logistic_model <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span><span class='va'>logistic_model</span>, <span class='op'>~</span> <span class='fu'>broom</span><span class='fu'>::</span><span class='fu'><a href='https://generics.r-lib.org/reference/tidy.html'>tidy</a></span><span class='op'>(</span><span class='va'>.</span>, conf.int <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># unnest results</span>
<span class='va'>data_regression_analysis_cdi</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>unnest</span><span class='op'>(</span><span class='va'>logistic_model</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>dplyr</span><span class='fu'>::</span><span class='fu'><a href='https://dplyr.tidyverse.org/reference/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>term</span> <span class='op'>==</span> <span class='st'>"concentration"</span><span class='op'>)</span>

<span class='co'># exponentiate</span>
<span class='va'>data_regression_analysis_cdi</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis_cdi</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='va'>estimate</span>, <span class='va'>conf.low</span>, <span class='va'>conf.high</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/Log.html'>exp</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


We append the two analyses:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='fu'>bind_rows</span><span class='op'>(</span><span class='va'>data_regression_analysis_wo_cdi</span>, <span class='va'>data_regression_analysis_cdi</span><span class='op'>)</span>
</code></pre></div>

</div>


Graph:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># make the graph</span>
<span class='va'>graph_analysis_comorbidities_air_pollution</span> <span class='op'>&lt;-</span>
  <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>.</span>, <span class='fu'>aes</span><span class='op'>(</span>
    x <span class='op'>=</span> <span class='va'>estimate</span>,
    y <span class='op'>=</span> <span class='va'>pollutant</span>,
    xmin <span class='op'>=</span> <span class='va'>conf.low</span>,
    xmax <span class='op'>=</span> <span class='va'>conf.high</span>
  <span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_vline</span><span class='op'>(</span>xintercept <span class='op'>=</span> <span class='fl'>1</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_pointrange</span><span class='op'>(</span>size <span class='op'>=</span> <span class='fl'>1.2</span>, colour <span class='op'>=</span> <span class='st'>"deepskyblue3"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_x_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>5</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span><span class='op'>~</span> <span class='va'>comorbidities</span> , nrow <span class='op'>=</span> <span class='fl'>1</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>""</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Odds of Having the Comorbidity"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='va'>custom_theme</span> <span class='op'>+</span>
  <span class='fu'>theme</span><span class='op'>(</span>plot.margin <span class='op'>=</span> <span class='fu'>margin</span><span class='op'>(</span><span class='fl'>1</span>, <span class='fl'>1</span>, <span class='fl'>1</span>,<span class='op'>-</span><span class='fl'>1</span>, <span class='st'>"cm"</span><span class='op'>)</span><span class='op'>)</span>


<span class='co'># print the graph</span>
<span class='va'>graph_analysis_comorbidities_air_pollution</span>
</code></pre></div>
![](script_comorbidities_air_pollution_files/figure-html5/unnamed-chunk-11-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_analysis_comorbidities_air_pollution</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
    <span class='st'>"3.outputs"</span>,
    <span class='st'>"1.figures"</span>,
    <span class='st'>"2.study_comorbidites_long_term_air_pollution"</span>,
    <span class='st'>"graph_analysis_comorbidities_air_pollution.pdf"</span>
  <span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>50</span>,
  height <span class='op'>=</span> <span class='fl'>15</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>



Table:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># print the table</span>
<span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>dplyr</span><span class='fu'>::</span><span class='fu'><a href='https://dplyr.tidyverse.org/reference/select.html'>select</a></span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>comorbidities</span>, <span class='va'>estimate</span>, <span class='va'>conf.low</span>, <span class='va'>conf.high</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='va'>estimate</span><span class='op'>:</span><span class='va'>conf.high</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/Round.html'>round</a></span><span class='op'>(</span><span class='va'>.</span>, <span class='fl'>1</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename</span><span class='op'>(</span>
    <span class='st'>"Pollutant"</span> <span class='op'>=</span> <span class='va'>pollutant</span>,
    <span class='st'>"Comorbodities"</span> <span class='op'>=</span> <span class='va'>comorbidities</span>,
    <span class='st'>"Point Estimate"</span> <span class='op'>=</span> <span class='va'>estimate</span>,
    <span class='st'>"Lower Bound 95% CI"</span> <span class='op'>=</span> <span class='va'>conf.low</span>,
    <span class='st'>"Upper Bound 95% CI"</span> <span class='op'>=</span> <span class='va'>conf.high</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/pkg/knitr/man/kable.html'>kable</a></span><span class='op'>(</span>align <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"l"</span>, <span class='st'>"l"</span>, <span class='fu'><a href='https://rdrr.io/r/base/rep.html'>rep</a></span><span class='op'>(</span><span class='st'>"c"</span>, <span class='fl'>3</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>


|Pollutant              |Comorbodities              | Point Estimate | Lower Bound 95% CI | Upper Bound 95% CI |
|:----------------------|:--------------------------|:--------------:|:------------------:|:------------------:|
|NO2 Exposure (µg/m³)   |Sex                        |      0.9       |        0.9         |        1.0         |
|NO2 Exposure (µg/m³)   |HBP                        |      1.0       |        0.9         |        1.1         |
|NO2 Exposure (µg/m³)   |Diabetes                   |      1.0       |        0.9         |        1.1         |
|NO2 Exposure (µg/m³)   |COPD                       |      1.1       |        1.0         |        1.2         |
|NO2 Exposure (µg/m³)   |Renal Failure              |      1.1       |        1.0         |        1.2         |
|NO2 Exposure (µg/m³)   |Cancer                     |      1.1       |        1.0         |        1.1         |
|O3 Exposure (µg/m³)    |Sex                        |      1.1       |        1.0         |        1.1         |
|O3 Exposure (µg/m³)    |HBP                        |      1.0       |        0.9         |        1.1         |
|O3 Exposure (µg/m³)    |Diabetes                   |      1.0       |        0.9         |        1.1         |
|O3 Exposure (µg/m³)    |COPD                       |      0.9       |        0.8         |        1.0         |
|O3 Exposure (µg/m³)    |Renal Failure              |      0.9       |        0.8         |        1.0         |
|O3 Exposure (µg/m³)    |Cancer                     |      1.0       |        0.9         |        1.1         |
|PM10 Exposure (µg/m³)  |Sex                        |      0.9       |        0.7         |        1.2         |
|PM10 Exposure (µg/m³)  |HBP                        |      1.0       |        0.8         |        1.3         |
|PM10 Exposure (µg/m³)  |Diabetes                   |      1.0       |        0.8         |        1.3         |
|PM10 Exposure (µg/m³)  |COPD                       |      1.3       |        0.9         |        1.8         |
|PM10 Exposure (µg/m³)  |Renal Failure              |      1.1       |        0.9         |        1.4         |
|PM10 Exposure (µg/m³)  |Cancer                     |      1.5       |        1.1         |        2.1         |
|PM2.5 Exposure (µg/m³) |Sex                        |      0.9       |        0.6         |        1.2         |
|PM2.5 Exposure (µg/m³) |HBP                        |      1.0       |        0.7         |        1.3         |
|PM2.5 Exposure (µg/m³) |Diabetes                   |      1.0       |        0.7         |        1.4         |
|PM2.5 Exposure (µg/m³) |COPD                       |      1.4       |        0.9         |        2.2         |
|PM2.5 Exposure (µg/m³) |Renal Failure              |      1.2       |        0.8         |        1.7         |
|PM2.5 Exposure (µg/m³) |Cancer                     |      1.5       |        1.1         |        2.3         |
|SO2 Exposure (µg/m³)   |Sex                        |      0.5       |        0.2         |        1.1         |
|SO2 Exposure (µg/m³)   |HBP                        |      1.4       |        0.6         |        3.1         |
|SO2 Exposure (µg/m³)   |Diabetes                   |      1.6       |        0.7         |        3.7         |
|SO2 Exposure (µg/m³)   |COPD                       |      2.1       |        0.7         |        6.1         |
|SO2 Exposure (µg/m³)   |Renal Failure              |      2.3       |        1.0         |        5.2         |
|SO2 Exposure (µg/m³)   |Cancer                     |      1.8       |        0.8         |        4.5         |
|NO2 Exposure (µg/m³)   |Cardiac Disease Infarction |      0.9       |        0.9         |        1.0         |
|O3 Exposure (µg/m³)    |Cardiac Disease Infarction |      1.1       |        1.0         |        1.2         |
|PM10 Exposure (µg/m³)  |Cardiac Disease Infarction |      0.8       |        0.6         |        1.0         |
|PM2.5 Exposure (µg/m³) |Cardiac Disease Infarction |      0.6       |        0.4         |        1.0         |
|SO2 Exposure (µg/m³)   |Cardiac Disease Infarction |      0.6       |        0.2         |        1.4         |

</div>


```{.r .distill-force-highlighting-css}
```
